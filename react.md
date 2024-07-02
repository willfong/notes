
[https://tailwindcss.com/docs/guides/create-react-app](https://tailwindcss.com/docs/guides/create-react-app)

## Setup
  
```bash
npx create-react-app my-app-name

npm install react-router-dom date-fns date-fns-tz
npm install idb

npm install -D tailwindcss @tailwindcss/forms @headlessui/react @heroicons/react @babel/plugin-proposal-private-property-in-object


npx tailwindcss init

rm public/favicon.ico public/logo* src/App.css src/App.test.js src/logo.svg src/setupTests.js
```

yarn
```bash
yarn create react-app my-app-name
yarn add tailwindcss --dev
yarn tailwindcss init

yarn add react-router-dom date-fns idb

yarn add -D @tailwindcss/forms @headlessui/react @heroicons/react @babel/plugin-proposal-private-property-in-object
```


## `package.json`


```json
"proxy": "http://localhost:3000",
```


## `tailwind.config.js`
  

```javascript
/** @type {import('tailwindcss').Config} */

module.exports = {
  content: ["./src/**/*.{js,jsx,ts,tsx}","./public/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [require("@tailwindcss/forms")],
};
```


## `index.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
  
## `public/index.html`

```html
<html lang="en" class="h-full bg-white">
...
<body class="h-full">
```


## Testing `App.js`

```javascript
export default function App() {
  return <h1 className="text-2xl p-4 text-blue-500">Hello World</h1>;
}
```
  
## src/App.js

```javascript  
import React, { useState, useEffect } from "react";
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";

import * as API from "./api";

import Header from "./components/header";
import Home from "./pages/home";
import CreateAccount from "./pages/create-account";
import Login from "./pages/login";
import Settings from "./pages/settings";

export default function App() {
  const [jwt, setJwt] = useState(false);
  useEffect(() => {
    const handleJwt = async () => {
      const result = await API.getJwt();
      if (result) {
        setJwt(result);
      }
    };
    handleJwt();
  }, []);

  const handleLogout = () => {
    API.logout();
    setJwt(false);
  };

  const handleLogin = async (loginCode) => {
    const jwt = await API.login(loginCode);
    if (jwt) {
      setJwt(jwt);
      return true;
    }

    return false;
  };

  const routeList = [
    {
      name: "Home",
      path: "/",
      component: Home,
      nav: true,
      authRequired: false,
    },
    {
      name: "Create Account",
      path: "/create-account",
      component: CreateAccount,
      nav: true,
      authRequired: false,
    },
    {
      name: "Login",
      path: "/login",
      component: Login,
      nav: true,
      authRequired: false,
      props: { handleLogin },
    },
    {
      name: "Settings",
      path: "/settings",
      component: Settings,
      nav: true,
      authRequired: true,
      props: { jwt },
    },
  ];

  return (
    <Router>
      <div className="min-h-full flex flex-col">
        <Header
          routeList={routeList}
          handleLogout={handleLogout}
          isLoggedIn={jwt ? true : false}
        />
        <div>
          <Routes>
            {routeList.map((route) => (
              <Route
                key={route.path}
                path={route.path}
                element={<route.component {...route.props} />}
              />
            ))}
          </Routes>
        </div>
      </div>
    </Router>
  );
}
```



## src/api.js
```javascript
const isValidJwt = (jwt) => {
  const payload = JSON.parse(atob(jwt.split(".")[1]));
  const now = Date.now() / 1000;
  if (payload.exp < now) {
    return false;
  }
  return true;
};

export const getJwt = async () => {
  const jwt = localStorage.getItem("jwt");
  if (!jwt) {
    return false;
  }
  if (!isValidJwt(jwt)) {
    const loginCode = localStorage.getItem("loginCode");
    const error = await login(loginCode);
    if (error) {
      return false;
    }
    return localStorage.getItem("jwt");
  }
  return jwt;
};

export const createAccount = async () => {
  try {
    const response = await fetch("/auth/create-new-user");
    const data = await response.json();

    if (data.loginCode) {
      //localStorage.setItem("jwt", data.token);
      //localStorage.setItem("loginCode", data.loginCode);
      return data.loginCode;
    } else {
      return false;
    }
  } catch (error) {
    console.error("There was an error logging in:", error);
    return false;
  }
};

export const login = async (loginCode) => {
  try {
    const response = await fetch("/auth/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ code: loginCode }),
    });

    const data = await response.json();

    if (data) {
      localStorage.setItem("jwt", data.token);
      localStorage.setItem("loginCode", data.loginCode);
      return data.token;
    } else {
      return false;
    }
  } catch (error) {
    console.error("There was an error logging in:", error);
    return false;
  }
};

export const logout = () => {
  localStorage.removeItem("jwt");
  localStorage.removeItem("loginCode");
};

export const fetchWords = async (jwt) => {
  try {
    const response = await fetch("/api/words", {
      headers: {
        Authorization: `Bearer ${jwt}`,
      },
    });

    const result = await response.json();

    if (result.error) {
      return [result.error, false];
    } else {
      return [
        false,
        result.results.sort((a, b) => a.word.localeCompare(b.word)),
      ];
    }
  } catch (err) {
    console.error(err);
    return ["An error occurred. Please try again later.", false];
  }
};

export const addWords = async (jwt, formData) => {
  try {
    const response = await fetch("/api/words-add", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${jwt}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        word: formData.word.trim(),
        translation: formData.translation.trim(),
        literal_translation: formData.literalTranslation.trim(),
        notes: formData.notes.trim(),
      }),
    });

    const result = await response.json();

    if (result.error) {
      return [result.error, false];
    } else {
      return [
        false,
        result.results.sort((a, b) => a.word.localeCompare(b.word)),
      ];
    }
  } catch (err) {
    console.error(err);
    return ["An error occurred. Please try again later.", false];
  }
};
```

## pages/home.js

```javascript
import React, { useState, useEffect } from "react";
import { useNavigate } from "react-router-dom";

import * as API from "../api";

function Home() {
  const [checkedLogin, setCheckedLogin] = useState(false);
  useEffect(() => {
    const isLoggedIn = async () => {
      const jwt = await API.getJwt();
      if (jwt) {
        navigate("/settings");
      }
      setCheckedLogin(true);
    };
    isLoggedIn();
  }, []);

  const navigate = useNavigate();

  return (
    <div className="App px-8">
      <section className="max-w-4xl mx-auto mt-6">
        <h1 className="text-2xl text-slate-600 font-semibold my-4">
          Welcome &#x1F60A;
        </h1>
        <p className="text-slate-500">This is img.gw</p>
        {!checkedLogin && (
          <p className="text-slate-50">
            Checking logged in status... one moment please...
          </p>
        )}
      </section>
    </div>
  );
}

export default Home;
```

## pages/create-account
```javascript
import React, { useState } from "react";
import * as API from "../api";

function CreateAccount() {
  const [data, setData] = useState("");

  const getUser = async () => {
    const data = await API.createAccount();
    setData(data);
  };

  return (
    <div className="App px-8">
      <section className="max-w-4xl mx-auto mt-6">
        <h1 className="text-2xl text-slate-600 font-semibold my-4">
          Create an account
        </h1>
        <p className="text-slate-500">
          When you create an account, you'll be able to save your words. We
          don't collect any of your personal information, so you just need to
          get a special code. We recommend to save it to your password manager
          to make it easier to retrieve. If you lose your code, you won't be
          able to access your data. Keep it safe!
        </p>
        <button
          className="bg-amber-400 text-gray-700 px-4 py-2 rounded my-4"
          onClick={getUser}
        >
          Get a login code
        </button>
        {data && (
          <>
            <p className="text-slate-500 mt-4">Your login code is: {data}</p>
            <button
              className="bg-amber-400 text-gray-700 px-4 py-2 rounded my-4"
              onClick={() => {
                navigator.clipboard.writeText(data);
              }}
            >
              Copy to Clipboard
            </button>
          </>
        )}
      </section>
    </div>
  );
}

export default CreateAccount;
```


## pages/login.js
```javascript
import React, { useState } from "react";
import { useNavigate } from "react-router-dom";

function Login({ handleLogin }) {
  const [loginCode, setLoginCode] = useState("");
  const [message, setMessage] = useState("");

  const navigate = useNavigate();

  const handleSubmit = async (event) => {
    event.preventDefault();
    try {
      const results = await handleLogin(loginCode);
      if (results) {
        navigate("/settings");
      } else {
        setMessage("There was an error logging in");
      }
    } catch (error) {
      console.error("There was an error logging in:", error);
      setMessage("There was an error logging in.");
    }
  };

  return (
    <div className="App px-8">
      <section className="max-w-4xl mx-auto mt-6">
        <h1 className="text-2xl text-slate-600 font-semibold my-4">Login</h1>
        <p className="text-slate-500">Enter your login code below.</p>
        <form onSubmit={handleSubmit}>
          <input
            name="password"
            type="password"
            placeholder="Enter login code"
            value={loginCode}
            onChange={(e) => setLoginCode(e.target.value)}
            className="p-2 mt-4 border rounded"
          />
          <button
            type="submit"
            className="ml-2 px-4 py-2 bg-amber-400 text-white rounded"
          >
            Go!
          </button>
        </form>
        {message && <p className="mt-4">{message}</p>}
      </section>
    </div>
  );
}

export default Login;
```


## pages/settings.js

```javascript
import React from "react";

function Settings() {
  return (
    <div className="App px-8">
      <section className="max-w-4xl mx-auto mt-6">
        <h1 className="text-2xl text-slate-600 font-semibold my-4">Settings</h1>
      </section>
    </div>
  );
}

export default Settings;
```

## components/header.js

```javascript
import { useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import { Dialog } from "@headlessui/react";
import { Bars3Icon, XMarkIcon } from "@heroicons/react/24/outline";

import ImaginationNetworkLogo from "./logo"; // Adjust the path as necessary

export default function Header({ routeList, logout, isLoggedIn }) {
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);
  const navigate = useNavigate();

  const handleLogout = async () => {
    logout();
    navigate("/");
  };

  return (
    <header className="bg-white">
      <nav
        className="mx-auto flex max-w-7xl items-center justify-between p-6 lg:px-8"
        aria-label="Global"
      >
        <a href="#" className="-m-1.5 p-1.5">
          <span className="sr-only">img.in.net</span>
          <ImaginationNetworkLogo size="50" />
          img.in.net {isLoggedIn ? "🔒" : ""}
        </a>
        <div className="flex lg:hidden">
          <button
            type="button"
            className="-m-2.5 inline-flex items-center justify-center rounded-md p-2.5 text-gray-700"
            onClick={() => setMobileMenuOpen(true)}
          >
            <span className="sr-only">Open main menu</span>
            <Bars3Icon className="h-6 w-6" aria-hidden="true" />
          </button>
        </div>
        <div className="hidden lg:flex lg:gap-x-12">
          {routeList
            .filter((item) => item.authRequired === isLoggedIn)
            .map((item) => (
              <Link
                key={item.name}
                to={item.path}
                className="text-sm font-semibold leading-6 text-gray-900"
              >
                {item.name}
              </Link>
            ))}
          {isLoggedIn && (
            <button
              type="button"
              className="text-sm font-semibold leading-6 text-gray-900"
              onClick={handleLogout}
            >
              Logout
            </button>
          )}
        </div>
      </nav>
      <Dialog
        as="div"
        className="lg:hidden"
        open={mobileMenuOpen}
        onClose={setMobileMenuOpen}
      >
        <div className="fixed inset-0 z-10" />
        <Dialog.Panel className="fixed inset-y-0 right-0 z-10 w-full overflow-y-auto bg-white px-6 py-6 sm:max-w-sm sm:ring-1 sm:ring-gray-900/10">
          <div className="flex items-center justify-between">
            <a href="#" className="-m-1.5 p-1.5">
              <span className="sr-only">Your Company</span>
              <img
                className="h-8 w-auto"
                src="https://tailwindui.com/img/logos/mark.svg?color=indigo&shade=600"
                alt=""
              />
            </a>
            <button
              type="button"
              className="-m-2.5 rounded-md p-2.5 text-gray-700"
              onClick={() => setMobileMenuOpen(false)}
            >
              <span className="sr-only">Close menu</span>
              <XMarkIcon className="h-6 w-6" aria-hidden="true" />
            </button>
          </div>
          <div className="mt-6 flow-root">
            <div className="-my-6 divide-y divide-gray-500/10">
              <div className="space-y-2 py-6">
                {routeList
                  .filter((item) => item.authRequired === isLoggedIn)
                  .map((item) => (
                    <Link
                      key={item.name}
                      to={item.path}
                      className="-mx-3 block rounded-lg px-3 py-2 text-base font-semibold leading-7 text-gray-900 hover:bg-gray-50"
                    >
                      {item.name}
                    </Link>
                  ))}
                {isLoggedIn && (
                  <button
                    type="button"
                    className="-mx-3 block rounded-lg px-3 py-2 text-base font-semibold leading-7 text-gray-900 hover:bg-gray-50"
                    onClick={handleLogout}
                  >
                    Logout
                  </button>
                )}
              </div>
            </div>
          </div>
        </Dialog.Panel>
      </Dialog>
    </header>
  );
}

```
## Old Data


import React from "react";

import React, { useState, useEffect } from "react";

import { Link } from "react-router-dom";

  

import SettingsCategoriesDetails from "./SettingsCategoriesDetail";

import { categoriesAddNew, categoriesGetAll } from "../idb";

  

  

---

**State**

  

const [newCategoryName, setNewCategoryName] = useState("");

  

UseEffect

  useEffect(() => {

    fetchCategories();

  }, []);

  

  

---

**API Calls**

  

import React, { useState, useEffect } from "react";

  

function Page({}) {

  const [data, setData] = useState([]);

  _const_ [loading, setLoading] = useState(true);

  _const_ [error, setError] = useState(null);

  

  useEffect(() => {

    handleGetData();

   }, []);

  

  const handleGetData = async () => {

    fetch("/api/get-data")

      .then((_response_) => {

        if (!response.ok) {

          throw new Error("Network response was not ok");

        }

        return response.json();

      })

      .then((_data_) => {

        setData(data);

        console.log(data);

        setLoading(false);

      })

      .catch((_error_) => {

        setError(error);

        setLoading(false);

      });

  }

  

  if (loading) return <div>Loading...</div>;

  if (error) return <div>Error: {error.message}</div>;

  

  return (

    <div>{/** Your data **/}</div>

  )

}

  

  

---

**Input**

  

_const_ [email, setEmail] = useState("");

  

  

  const handleNewCategoryNameChange = (event) => {

    setNewCategoryName(event.target.value);

  };

        <input

          className="p-2 w-full sm:w-48"

          type="text"

          placeholder="email"

          value={email}

          _onChange_={(_e_) => setEmail(e.target.value)}

        />

  

  

---

**IndexedDB**

  

import { openDB, deleteDB } from "idb";

  

const DB_NAME = "simplebudget";

const DB_VERSION = 2;

const TABLE_PURCHASES = "purchases";

  

async function initializeDB() {

return await openDB(DB_NAME, DB_VERSION, {

upgrade(db) {

if (!db.objectStoreNames.contains(TABLE_PURCHASES)) {

const tblPurchases = db.createObjectStore(TABLE_PURCHASES, { keyPath: "id", autoIncrement: true });

tblPurchases.createIndex(TABLE_PURCHASES_INDEX_DATE, "date", { unique: false });

tblPurchases.createIndex(TABLE_PURCHASES_INDEX_CATEGORYID, ["categoryId", "date"], { unique: false });

}

},

});

}

  

const dbPromise = initializeDB();

  

export async function categoriesAddNew(name, budget, monthly) {

const db = await dbPromise;

const tx = db.transaction(TABLE_CATEGORIES, "readwrite");

const store = tx.objectStore(TABLE_CATEGORIES);

await store.add({ name, budget: budget ? parseFloat(budget) : 0, monthly });

await tx.complete;

}

  

export async function categoriesGetAll() {

const db = await dbPromise;

const tx = db.transaction(TABLE_CATEGORIES, "readonly");

const store = tx.objectStore(TABLE_CATEGORIES);

return store.getAll();

}

  

  

---

**App**

  

import React from "react";

import { BrowserRouter as Router, Route, Routes } from "react-router-dom";

//import HomePage from "./pages/Home";

import SpendPage from "./pages/Spend";

import SpendDetailPage from "./pages/SpendDetail";

import ReportPage from "./pages/Report";

import PlanPage from "./pages/Plan";

import SettingsPage from "./pages/Settings";

import SettingsCategoriesUpdatePage from "./pages/SettingsCategoriesUpdate";

import Header from "./components/Header";

  

const routeList = [

{ name: "Home", path: "/", component: SpendPage, nav: true },

{ name: "Spend", path: "/spend", component: SpendPage, nav: true },

{

name: "SettingsCategoryUpdate",

path: "/settings/category-update/:id",

component: SettingsCategoriesUpdatePage,

nav: false,

},

];

  

export default function App() {

return (

<Router>

{/*

        This example requires updating your template:

  

        ```

        <html class="h-full">

        <body class="h-full">

        ```

      */}

<div className="min-h-full flex flex-col">

<Header routeList={routeList} />

<div>

<Routes>

{routeList.map((route) => (

<Route key={route.path} path={route.path} element={<route.component />} />

))}

</Routes>

</div>

</div>

</Router>

);

}

  

  

--- 

Buttons

  

[https://heroicons.com](https://heroicons.com)

  

document-magnifying-glass -> DocumentMagnifiyingGlassIcon

  

import {

  DocumentMagnifiyingGlassIcon,

} from "@heroicons/react/24/outline";
```
npx create-react-app my-app-name
npm install -D tailwindcss
npx tailwindcss init

npm install react-router-dom
npm install date-fns date-fns-tz
npm install idb 
npm install -D _@tailwindcss/forms_

npm install -D @headlessui/react

npm install -D @heroicons/react

npm install -D @babel/plugin-proposal-private-property-in-object

  

  

yarn create react-app my-app-name

yarn add tailwindcss --dev

yarn tailwindcss init

  

yarn add react-router-dom date-fns idb

yarn add -D @tailwindcss/forms @headlessui/react @heroicons/react @babel/plugin-proposal-private-property-in-object

  

  

---

**Proxy**

  

In `package.json`:

"proxy": "http://localhost:5000" 

  

  

---

**Tailwind**

  

tailwind.config.js

  

_/** @type {import('tailwindcss').Config} */_

_module_._exports_ = {

  content: ["./src/**/*.{js,jsx,ts,tsx}"],

  theme: {

    extend: {},

  },

  plugins: [require("@tailwindcss/forms")],

};

  

  

index.css

  

@tailwind base;

@tailwind components;

@tailwind utilities;

  

  

---

**Test App.js**

  

export default _function_ App() {

  return <h1 _className_="text-2xl p-4 text-blue-500">Hello World</h1>;

}

  

  

---

**Headers**

  

import React from "react";

import React, { useState, useEffect } from "react";

import { Link } from "react-router-dom";

  

import SettingsCategoriesDetails from "./SettingsCategoriesDetail";

import { categoriesAddNew, categoriesGetAll } from "../idb";

  

  

---

**State**

  

const [newCategoryName, setNewCategoryName] = useState("");

  

UseEffect

  useEffect(() => {

    fetchCategories();

  }, []);

  

  

---

**API Calls**

  

import React, { useState, useEffect } from "react";

  

function Page({}) {

  const [data, setData] = useState([]);

  _const_ [loading, setLoading] = useState(true);

  _const_ [error, setError] = useState(null);

  

  useEffect(() => {

    handleGetData();

   }, []);

  

  const handleGetData = async () => {

    fetch("/api/get-data")

      .then((_response_) => {

        if (!response.ok) {

          throw new Error("Network response was not ok");

        }

        return response.json();

      })

      .then((_data_) => {

        setData(data);

        console.log(data);

        setLoading(false);

      })

      .catch((_error_) => {

        setError(error);

        setLoading(false);

      });

  }

  

  if (loading) return <div>Loading...</div>;

  if (error) return <div>Error: {error.message}</div>;

  

  return (

    <div>{/** Your data **/}</div>

  )

}

  

  

---

**Input**

  

_const_ [email, setEmail] = useState("");

  

  

  const handleNewCategoryNameChange = (event) => {

    setNewCategoryName(event.target.value);

  };

        <input

          className="p-2 w-full sm:w-48"

          type="text"

          placeholder="email"

          value={email}

          _onChange_={(_e_) => setEmail(e.target.value)}

        />

  

  

---

**IndexedDB**

  

import { openDB, deleteDB } from "idb";

  

const DB_NAME = "simplebudget";

const DB_VERSION = 2;

const TABLE_PURCHASES = "purchases";

  

async function initializeDB() {

return await openDB(DB_NAME, DB_VERSION, {

upgrade(db) {

if (!db.objectStoreNames.contains(TABLE_PURCHASES)) {

const tblPurchases = db.createObjectStore(TABLE_PURCHASES, { keyPath: "id", autoIncrement: true });

tblPurchases.createIndex(TABLE_PURCHASES_INDEX_DATE, "date", { unique: false });

tblPurchases.createIndex(TABLE_PURCHASES_INDEX_CATEGORYID, ["categoryId", "date"], { unique: false });

}

},

});

}

  

const dbPromise = initializeDB();

  

export async function categoriesAddNew(name, budget, monthly) {

const db = await dbPromise;

const tx = db.transaction(TABLE_CATEGORIES, "readwrite");

const store = tx.objectStore(TABLE_CATEGORIES);

await store.add({ name, budget: budget ? parseFloat(budget) : 0, monthly });

await tx.complete;

}

  

export async function categoriesGetAll() {

const db = await dbPromise;

const tx = db.transaction(TABLE_CATEGORIES, "readonly");

const store = tx.objectStore(TABLE_CATEGORIES);

return store.getAll();

}

  

  

---

**App**

  

import React from "react";

import { BrowserRouter as Router, Route, Routes } from "react-router-dom";

//import HomePage from "./pages/Home";

import SpendPage from "./pages/Spend";

import SpendDetailPage from "./pages/SpendDetail";

import ReportPage from "./pages/Report";

import PlanPage from "./pages/Plan";

import SettingsPage from "./pages/Settings";

import SettingsCategoriesUpdatePage from "./pages/SettingsCategoriesUpdate";

import Header from "./components/Header";

  

const routeList = [

{ name: "Home", path: "/", component: SpendPage, nav: true },

{ name: "Spend", path: "/spend", component: SpendPage, nav: true },

{

name: "SettingsCategoryUpdate",

path: "/settings/category-update/:id",

component: SettingsCategoriesUpdatePage,

nav: false,

},

];

  

export default function App() {

return (

<Router>

{/*

        This example requires updating your template:

  

        ```

        <html class="h-full">

        <body class="h-full">

        ```

      */}

<div className="min-h-full flex flex-col">

<Header routeList={routeList} />

<div>

<Routes>

{routeList.map((route) => (

<Route key={route.path} path={route.path} element={<route.component />} />

))}

</Routes>

</div>

</div>

</Router>

);

}

  

  

--- 

Buttons

  

[https://heroicons.com](https://heroicons.com)

  

document-magnifying-glass -> DocumentMagnifiyingGlassIcon

  

import {

  DocumentMagnifiyingGlassIcon,

} from "@heroicons/react/24/outline";
