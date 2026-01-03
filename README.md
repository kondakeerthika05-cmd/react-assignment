# react-assignment
Step 1 — Create Project
npx create-react-app restaurant-app
cd restaurant-app
npm start

✅ Step 2 — Install React Router
npm install react-router-dom

✅ Step 3 — Create Folder Structure

Inside src/, create:

src/
 ├─ pages/
 │   ├─ Login.jsx
 │   ├─ AdminDashboard.jsx
 │   ├─ CustomerDashboard.jsx
 │   ├─ UpdateRestaurant.jsx
 ├─ components/
 │   ├─ RestaurantCard.jsx
 ├─ context/
 │   ├─ AuthContext.jsx
 ├─ routes/
 │   ├─ PrivateRoute.jsx
 ├─ App.jsx

✅ Step 4 — Auth Context (Login + Role)

📄 src/context/AuthContext.jsx

import { createContext, useState, useEffect } from "react";

export const AuthContext = createContext();

export default function AuthContextProvider({ children }) {
  const [user, setUser] = useState(
    JSON.parse(localStorage.getItem("user")) || null
  );

  const login = (email, password) => {
    if (email === "admin@gmail.com" && password === "admin1234") {
      const data = { role: "admin" };
      setUser(data);
      localStorage.setItem("user", JSON.stringify(data));
      return "admin";
    }

    if (email === "customer@gmail.com" && password === "customer1234") {
      const data = { role: "customer" };
      setUser(data);
      localStorage.setItem("user", JSON.stringify(data));
      return "customer";
    }

    return "error";
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem("user");
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

✅ Step 5 — Login Page

📄 src/pages/Login.jsx

import { useContext, useState } from "react";
import { AuthContext } from "../context/AuthContext";
import { useNavigate } from "react-router-dom";

export default function Login() {
  const { login } = useContext(AuthContext);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const navigate = useNavigate();

  function handleSubmit(e) {
    e.preventDefault();

    const res = login(email, password);

    if (res === "admin") navigate("/admin/dashboard");
    else if (res === "customer") navigate("/customers/dashboard");
    else alert("Invalid credentials");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="email" onChange={(e) => setEmail(e.target.value)} />
      <input
        placeholder="password"
        type="password"
        onChange={(e) => setPassword(e.target.value)}
      />
      <button>Login</button>
    </form>
  );
}

✅ Step 6 — Private Route

Only logged-in users should access dashboards.

📄 src/routes/PrivateRoute.jsx

import { useContext } from "react";
import { Navigate } from "react-router-dom";
import { AuthContext } from "../context/AuthContext";

export default function PrivateRoute({ children }) {
  const { user } = useContext(AuthContext);

  if (!user) return <Navigate to="/" />;

  return children;
}

✅ Step 7 — Add Routes

📄 App.jsx

import { BrowserRouter, Routes, Route } from "react-router-dom";
import Login from "./pages/Login";
import AdminDashboard from "./pages/AdminDashboard";
import CustomerDashboard from "./pages/CustomerDashboard";
import PrivateRoute from "./routes/PrivateRoute";
import AuthContextProvider from "./context/AuthContext";

function App() {
  return (
    <AuthContextProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<Login />} />

          <Route
            path="/admin/dashboard"
            element={
              <PrivateRoute>
                <AdminDashboard />
              </PrivateRoute>
            }
          />

          <Route
            path="/customers/dashboard"
            element={
              <PrivateRoute>
                <CustomerDashboard />
              </PrivateRoute>
            }
          />
        </Routes>
      </BrowserRouter>
    </AuthContextProvider>
  );
}

export default App;
