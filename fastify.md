## Getting Started

```bash
npm init -y
npm install fastify @fastify/static @fastify/jwt bcrypt mariadb
npm install --save-dev nodemon dotenv
```

## `package.json` Updates

```json
"scripts": {
	"start": "node index.mjs",
	"dev": "nodemon -r dotenv/config index.mjs"
},
```

## `.env` Updates

```env
JWT_SECRET=secret
MARIADB_HOST=127.0.0.1
MARIADB_PORT=3306
MARIADB_USER=root
MARIADB_PASSWORD=
MARIADB_DATABASE=test
```



## `index.mjs`

```javascript
import Fastify from "fastify";
import staticPlugin from "@fastify/static";
import jwt from "@fastify/jwt";
import path from "path";
import { fileURLToPath } from "url";
import { dirname } from "path";

import authRoutes from "./auth.mjs";
import apiRoutes from "./api.mjs";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const fastify = Fastify({ logger: true });

fastify.register(jwt, {
	secret: process.env.JWT_SECRET,
});

fastify.decorate("authenticate", async (request, reply) => {
	try {
		await request.jwtVerify();
		const user = request.user;
        request.userDetails = user;
	} catch (err) {
		reply.code(403).send(err.message);
	}
});

fastify.register(staticPlugin, {
	root: path.join(__dirname, "static"),
	prefix: "/",
});

fastify.register(authRoutes, { prefix: "/auth" });
fastify.register(apiRoutes, { prefix: "/api" });

const start = async () => {
	try {
		await fastify.listen({ port: 3000, host: "0.0.0.0" });
		fastify.log.info(`Server listening on ${fastify.server.address().port}`);
	} catch (err) {
		fastify.log.error(err);
		process.exit(1);
	}
};

start();
```

## `API`

```javascript
import * as DB from "./db.mjs";

async function routes(fastify, options) {
  fastify.get("/users", {
    preValidation: [fastify.authenticate],
    handler: async (request, reply) => {
      const userDetails = request.userDetails;
      try {
        const query = "SELECT * FROM users";
        const results = await DB.query(query);
        reply.send(results);
      } catch (err) {
        reply.send(err);
      }
    },
  });

// Fix this POST req
	fastify.post("/users", async (request, reply) => {
		const { name, password } = request.body;
		if (!name || !password) {
			return reply.status(400).send({ error: "Missing required fields" });
		}

		try {
			const query = "INSERT INTO users (name, password) VALUES (?, ?)";
			const params = [name, password];
			const result = await DB.query(query, params);
			reply.send({
				success: true,
				message: "User created",
			});
		} catch (err) {
			reply.send(err);
		}
	});
}

export default routes;
```
  

## `DB`

```javascript
import MariaDB from "mariadb";

const pool = MariaDB.createPool({
  host: process.env.MARIADB_HOST,
  user: process.env.MARIADB_USER,
  password: process.env.MARIADB_PASSWORD,
  database: process.env.MARIADB_DATABASE,
  insertIdAsNumber: false, // this is important for the lastInsertId() method
});

export const query = async (query, params) => {
  let conn;
  try {
    conn = await pool.getConnection();
    const rows = await conn.query(query, params);
    return rows;
  } catch (err) {
    throw err;
  } finally {
    if (conn) conn.end();
  }
};

export const auth_create_new_user = async (hashedPassword) => {
  const sql = "INSERT INTO users (login_code, created_at) VALUES (?, NOW())";
  const result = await query(sql, [hashedPassword]);
  return result;
};

export const auth_me = async (id) => {
  const sql = "SELECT * FROM users WHERE id = ? LIMIT 1";
  const results = await query(sql, [id]);
  return results;
};

export const auth_login = async (id) => {
  const sql = "SELECT * FROM users WHERE id = ?";
  const result = await query(sql, [id]);
  return result;
};

export const auth_lastlogin = async (id) => {
  const sql = "UPDATE users SET last_login = NOW() WHERE id = ?";
  await query(sql, [id]);
};



```

## `Auth`

```javascript
import bcrypt from "bcrypt";
import crypto from "crypto";

import * as DB from "./db.mjs";

const generateRandomPassword = (length) => {
  const characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()_+";
  let password = "";
  const charactersLength = characters.length;
  
  for (let i = 0; i < length; i++) {
    const randomValue = crypto.randomInt(charactersLength);
   password += characters[randomValue];
  }
  
  return password;
};

const hashPassword = async (password) => {
  const saltRounds = 10;
  const hash = await bcrypt.hash(password, saltRounds);
  return hash;
};

const verifyPassword = async (userPassword, hashedPassword) => {
  try {
    const match = await bcrypt.compare(userPassword, hashedPassword);
    return match;
  } catch (error) {
    console.error(error);
  }
};

async function routes(fastify, options) {
  fastify.get("/create-new-user", async (request, reply) => {
    try {
      const newPassword = generateRandomPassword(32);
      const hashedPassword = await hashPassword(newPassword);
      const result = await DB.auth_create_new_user(hashedPassword);
      const code = `${newPassword}.${result.insertId}`;
      return reply.send({ code });
    } catch (err) {
      reply.send(err);
    }
  });

  fastify.get("/me", {
    preValidation: [fastify.authenticate],
    handler: async (request, reply) => {
      const userDetails = request.userDetails;
      try {
        const results = await DB.auth_me(userDetails.id);
        // Actually, the below might not need this, should never happen
        // since it came from JWT
        if (results.length === 0) {
          return reply.status(404).send({ error: "User not found." });
        }
        reply.send(results[0]);
      } catch (err) {
        reply.send(err);
      }
    },
  });

  fastify.post("/login", async (request, reply) => {
    const { code } = request.body;
    if (!code) {
      return reply.status(400).send({ error: "Missing code" });
    }

    const [userPassword, id] = code.split(".");
    try {
      const result = await DB.auth_login(id);
      if (result.length === 0) {
        return reply.status(400).send({ error: "Invalid credentials." });
      }

      if (await verifyPassword(userPassword, result[0].login_code)) {
        await DB.auth_lastlogin(id);
        reply.send({
          token: fastify.jwt.sign({ id: result[0].id }),
        });
      } else {
        return reply.status(400).send({ error: "Invalid credentials..." });
      }
    } catch (err) {
      reply.status(500).send(err);
    }
  });
}

export default routes;
```


## Schema

```sql
DROP TABLE IF EXISTS users;

CREATE TABLE `users` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`login_code` varchar(60) DEFAULT NULL,
`created_at` datetime DEFAULT current_timestamp(),
`last_login` datetime DEFAULT current_timestamp(),
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
CREATE UNIQUE INDEX login_code ON users (login_code);
```

## Testing Endpoint

Use the automated script:
```javascript
const http = require("http");

// Function to make a GET request
function getRequest(url, headers) {
  return new Promise((resolve, reject) => {
    const options = {
      hostname: "localhost",
      port: 3000,
      path: url,
      method: "GET",
      headers: headers,
    };

    const req = http.request(options, (res) => {
      let data = "";
      res.on("data", (chunk) => {
        data += chunk;
      });
      res.on("end", () => {
        resolve(JSON.parse(data));
      });
    });

    req.on("error", (error) => {
      reject(error);
    });

    req.end();
  });
}

// Function to make a POST request
function postRequest(url, data) {
  return new Promise((resolve, reject) => {
    const options = {
      hostname: "localhost",
      port: 3000,
      path: url,
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Content-Length": Buffer.byteLength(JSON.stringify(data)),
      },
    };

    const req = http.request(options, (res) => {
      let responseData = "";
      res.on("data", (chunk) => {
        responseData += chunk;
      });
      res.on("end", () => {
        resolve(JSON.parse(responseData));
      });
    });

    req.on("error", (error) => {
      reject(error);
    });

    req.write(JSON.stringify(data));
    req.end();
  });
}

async function main() {
  try {
    // Step 1: Make API GET to create-new-user
    console.log("Creating new user...");
    const createUserResponse = await getRequest(
      "http://localhost:3000/auth/create-new-user"
    );
    const code = createUserResponse.code;
    console.log("Login Code:", createUserResponse);

    // Pausing for 5 seconds to allow the server to update the last_login field
    console.log("Pausing for 5 seconds...");
    await new Promise((resolve) => setTimeout(resolve, 5000));

    // Step 2: Make API POST to login with loginCode
    console.log("Logging in to get JWT...");
    const loginResponse = await postRequest("/auth/login", { code });
    const token = loginResponse.token;
    console.log("Token:", token);

    // Step 3: Make API GET to me endpoint with token
    console.log("Getting user info...");
    const meResponse = await getRequest("/auth/me", {
      Authorization: `Bearer ${token}`,
    });

    // Step 4: Print the results
    console.log("User Info:", meResponse);
  } catch (error) {
    console.error("Error:", error.message);
  }
}

main();
```

Or you can do it manually:

Creating a new user  
```bash
curl localhost:3000/auth/create-new-user
```

Result:
```json
{"code":"#mw)*VUREz9D&Z)6(6ruyE!P*F2g6F9j.11"}
```

Logging in to get JWT

```bash
curl -X POST localhost:3000/auth/login -H "Content-Type: application/json" -d '{"code":"#mw)*VUREz9D&Z)6(6ruyE!P*F2g6F9j.11"}'
```

Result:
```json
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTEsImlhdCI6MTcwMzQzMDc2Nn0.0ktUnUFPOB6kXc-gXzMiQivT76-EGR5u05jJhymRaAQ"}
```

Using the JWT for an API call
  
```bash
curl -X GET http://localhost:3000/auth/me -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTEsImlhdCI6MTcwMzQzMDc2Nn0.0ktUnUFPOB6kXc-gXzMiQivT76-EGR5u05jJhymRaAQ"
```

Result:
```json
[{"id":1,"login_code":"$2b$10$5aHWHFhywH3nApcZUbaIw.8CfzKfOj0YRR5XWw/8wT4ApeZe020zy","created_at":"2024-03-03T08:12:21.000Z","last_login":"2024-03-03T08:12:21.000Z"}]
```
