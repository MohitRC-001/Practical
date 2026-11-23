Question
1. Create an Express.js server for a Student Management System with the routes /, /students, /students/:id and /about.
Program
// server.js
const express = require("express");
const app = express();
const PORT = 3000;
 
const students = [
  { id: 1, name: "Aarav Patil", course: "MCA" },
  { id: 2, name: "Neha Sharma", course: "MCA" },
  { id: 3, name: "Rahul Deshmukh", course: "MBA" }
];
 
app.get("/", (req, res) => {
  res.send("Welcome to Student Management System");
});
 
app.get("/students", (req, res) => {
  res.json(students);
});
 
app.get("/students/:id", (req, res) => {
  const student = students.find(s => s.id === Number(req.params.id));
  if (!student) return res.status(404).send("Student not found");
  res.json(student);
});
 
app.get("/about", (req, res) => {
  res.send("D. Y. Patil Institute of MCA & Management, Pune");
});
 
app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
 
// Run:
// npm init -y
// npm install express
// node server.js
Output
Expected Output: Start the server and open http://localhost:3000. The home route displays the welcome message. /students displays all student records in JSON, /students/1 displays the selected student, and /about displays institute information.
 Question
2. Develop an Express.js application with custom middleware to display the current date, time and requested URL. Create a POST route to accept student name and email using JSON.
Program
// server.js
const express = require("express");
const app = express();
const PORT = 3000;
 
app.use(express.json());
 
// Custom middleware
app.use((req, res, next) => {
  console.log("Date and Time:", new Date().toLocaleString());
  console.log("Requested URL:", req.originalUrl);
  next();
});
 
app.get("/", (req, res) => {
  res.send("Middleware demonstration server");
});
 
app.post("/students", (req, res) => {
  const { name, email } = req.body;
  if (!name || !email) {
    return res.status(400).json({ message: "Name and email are required" });
  }
  res.status(201).json({
    message: "Student details received successfully",
    student: { name, email }
  });
});
 
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
 
// Run:
// npm init -y
// npm install express
// node server.js
//
// Test POST /students with JSON body:
// { "name": "Asha", "email": "asha@example.com" }
Output
Expected Output: Open the server in a browser or send a POST request to /students using Postman. The terminal displays the request date, time and URL. The POST response returns the submitted name and email in JSON.

 Question
3. Develop a simple login application using Express.js and JWT. Generate a JWT after successful login and protect a route using token verification.
Program
// server.js
const express = require("express");
const jwt = require("jsonwebtoken");
const app = express();
app.use(express.json());
 
const PORT = 3000;
const JWT_SECRET = "replace_this_with_a_long_secret";
const DEMO_USER = { username: "admin", password: "admin123" };
 
app.post("/login", (req, res) => {
  const { username, password } = req.body;
  if (username !== DEMO_USER.username || password !== DEMO_USER.password) {
    return res.status(401).json({ message: "Invalid username or password" });
  }
  const token = jwt.sign({ username }, JWT_SECRET, { expiresIn: "1h" });
  res.json({ message: "Login successful", token });
});
 
function verifyToken(req, res, next) {
  const authHeader = req.headers.authorization;
  const token = authHeader && authHeader.startsWith("Bearer ")
    ? authHeader.split(" ")[1] : null;
 
  if (!token) return res.status(401).json({ message: "Token required" });
 
  jwt.verify(token, JWT_SECRET, (err, decoded) => {
    if (err) return res.status(403).json({ message: "Invalid or expired token" });
    req.user = decoded;
    next();
  });
}
 
app.get("/dashboard", verifyToken, (req, res) => {
  res.json({ message: `Welcome ${req.user.username}. Protected route accessed.` });
});
 
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
 
// Run:
// npm init -y
// npm install express jsonwebtoken
// node server.js
//
// Login JSON: { "username": "admin", "password": "admin123" }
// Send returned token as: Authorization: Bearer <token>
Output
Expected Output: POST /login with the demo credentials to receive a JWT. Use the token in the Authorization header to access GET /dashboard. Without a valid token, the protected route returns an unauthorized or forbidden response. For real applications, store passwords securely and keep the secret in an environment variable.

 Question
4. Create a Student REST API using Express.js and MongoDB. Perform insert, display, update and delete operations using Mongoose.
Program
// server.js
const express = require("express");
const mongoose = require("mongoose");
const app = express();
app.use(express.json());
 
mongoose.connect("mongodb://127.0.0.1:27017/studentDB")
  .then(() => console.log("MongoDB connected"))
  .catch(err => console.error("MongoDB connection error:", err));
 
const studentSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true },
  course: { type: String, required: true }
});
const Student = mongoose.model("Student", studentSchema);
 
// Insert a student
app.post("/students", async (req, res) => {
  try {
    const student = await Student.create(req.body);
    res.status(201).json(student);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});
 
// Display all students
app.get("/students", async (req, res) => {
  try {
    res.json(await Student.find());
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
 
// Update student using MongoDB ID
app.put("/students/:id", async (req, res) => {
  try {
    const student = await Student.findByIdAndUpdate(
      req.params.id, req.body, { new: true, runValidators: true }
    );
    if (!student) return res.status(404).json({ message: "Student not found" });
    res.json(student);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});
 
// Delete student using MongoDB ID
app.delete("/students/:id", async (req, res) => {
  try {
    const student = await Student.findByIdAndDelete(req.params.id);
    if (!student) return res.status(404).json({ message: "Student not found" });
    res.json({ message: "Student deleted successfully" });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});
 
app.listen(3000, () => console.log("Server running on port 3000"));
 
// Run:
// npm init -y
// npm install express mongoose
// Start MongoDB, then run: node server.js
Output
Expected Output: Run the server after starting MongoDB. Use Postman to create a student with POST /students, view records with GET /students, update a record with PUT /students/:id and delete it with DELETE /students/:id. MongoDB stores the records in studentDB.

 Question
5. Create a React application using Vite to display student information. Create a reusable Student component and pass student name, roll number and course using props.
Program
// Create project:
// npm create vite@latest student-app -- --template react
// cd student-app
// npm install
// npm run dev
 
// src/Student.jsx
function Student({ name, rollNo, course }) {
  return (
    <div className="student">
      <h3>{name}</h3>
      <p>Roll Number: {rollNo}</p>
      <p>Course: {course}</p>
    </div>
  );
}
export default Student;
 
// src/App.jsx
import Student from "./Student";
import "./App.css";
 
function App() {
  const students = [
    { id: 1, name: "Asha Patil", rollNo: "101", course: "MCA" },
    { id: 2, name: "Rahul Shah", rollNo: "102", course: "MCA" },
    { id: 3, name: "Neha Joshi", rollNo: "103", course: "MCA" }
  ];
 
  return (
    <main>
      <h1>Student Information</h1>
      {students.map(student => (
        <Student key={student.id} {...student} />
      ))}
    </main>
  );
}
export default App;
 
// src/App.css
body { font-family: Arial, sans-serif; background: #f4f6f8; }
main { max-width: 650px; margin: 30px auto; padding: 20px; }
.student { background: white; padding: 14px; margin: 12px 0;
  border-radius: 8px; box-shadow: 0 1px 5px #ccc; }
Output
Expected Output: Run the Vite development server and open the displayed local URL. The page displays three student records. Each record is rendered by the reusable Student component using props.

 Question
6. Develop a React To-Do List using useState. Provide functionality to add a task, display all tasks and delete a selected task.
Program
// src/App.jsx
import { useState } from "react";
 
function App() {
  const [task, setTask] = useState("");
  const [tasks, setTasks] = useState([]);
 
  const addTask = (e) => {
    e.preventDefault();
    const value = task.trim();
    if (!value) return;
    setTasks([...tasks, { id: Date.now(), title: value }]);
    setTask("");
  };
 
  const deleteTask = (id) => {
    setTasks(tasks.filter(item => item.id !== id));
  };
 
  return (
    <div style={{ maxWidth: 500, margin: "40px auto", fontFamily: "Arial" }}>
      <h1>To-Do List</h1>
      <form onSubmit={addTask}>
        <input value={task} onChange={e => setTask(e.target.value)}
          placeholder="Enter a task" />
        <button type="submit">Add Task</button>
      </form>
      <ul>
        {tasks.map(item => (
          <li key={item.id}>
            {item.title}{" "}
            <button onClick={() => deleteTask(item.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
export default App;
 
// Run:
// npm create vite@latest todo-app -- --template react
// cd todo-app && npm install && npm run dev
Output
Expected Output: Enter a task and click Add Task. The task appears in the list. Click Delete beside a task to remove only that selected task. The task list is maintained using the useState hook.

 Question
7. Create a React application with Home, About, Contact and Login pages using React Router. Implement navigation without reloading the browser.
Program
// Create project and install router:
// npm create vite@latest router-app -- --template react
// cd router-app
// npm install
// npm install react-router-dom
 
// src/App.jsx
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";
 
function Home() { return <h2>Welcome to our website</h2>; }
function About() { return <h2>About: D. Y. Patil Institute of MCA & Management</h2>; }
function Contact() {
  return <div><h2>Contact Us</h2><p>Email: director@dypimca.ac.in</p>
    <p>Pune, Maharashtra</p></div>;
}
function Login() {
  return <div><h2>Login</h2>
    <form><input placeholder="Username" />
      <input type="password" placeholder="Password" />
      <button type="button">Login</button></form>
  </div>;
}
 
export default function App() {
  return (
    <BrowserRouter>
      <nav style={{ display: "flex", gap: 20, padding: 16 }}>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact</Link>
        <Link to="/login">Login</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="/login" element={<Login />} />
      </Routes>
    </BrowserRouter>
  );
}
 
// Run: npm run dev
Output
Expected Output: Run the React application. Click Home, About, Contact and Login in the navigation bar. Each page is displayed through React Router without a full browser reload.

 Question
8. Create a React frontend and Express backend. Develop an API to display Employee ID, Name and Department. Fetch employee data using Axios and display it in a React table.
Program
// BACKEND: server/server.js
const express = require("express");
const cors = require("cors");
const app = express();
app.use(cors());
app.use(express.json());
 
const employees = [
  { id: 1, name: "Amit Patil", department: "IT" },
  { id: 2, name: "Sneha Kulkarni", department: "HR" },
  { id: 3, name: "Rohan Mehta", department: "Finance" }
];
 
app.get("/api/employees", (req, res) => res.json(employees));
app.listen(5000, () => console.log("API running on port 5000"));
 
// Backend setup:
// cd server
// npm init -y
// npm install express cors
// node server.js
 
// FRONTEND: src/App.jsx
import { useEffect, useState } from "react";
import axios from "axios";
 
export default function App() {
  const [employees, setEmployees] = useState([]);
  const [error, setError] = useState("");
 
  useEffect(() => {
    axios.get("http://localhost:5000/api/employees")
      .then(response => setEmployees(response.data))
      .catch(() => setError("Unable to load employee records"));
  }, []);
 
  return (
    <main>
      <h1>Employee Details</h1>
      {error && <p>{error}</p>}
      <table border="1" cellPadding="10">
        <thead><tr><th>Employee ID</th><th>Name</th><th>Department</th></tr></thead>
        <tbody>
          {employees.map(emp => (
            <tr key={emp.id}>
              <td>{emp.id}</td><td>{emp.name}</td><td>{emp.department}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </main>
  );
}
 
// Frontend setup:
// npm install axios
Output
Expected Output: Start the Express backend on port 5000 and the React frontend using Vite. The React page sends a GET request through Axios and displays the three employee records in a table.

 Question
9. Develop a Product Management Application using React, Express.js and MongoDB. Implement add, display, update price and delete operations.
Program
// BACKEND: server/server.js
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const app = express();
app.use(cors());
app.use(express.json());
 
mongoose.connect("mongodb://127.0.0.1:27017/productDB")
  .then(() => console.log("MongoDB connected"))
  .catch(err => console.error(err));
 
const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  price: { type: Number, required: true, min: 0 }
});
const Product = mongoose.model("Product", productSchema);
 
app.post("/api/products", async (req, res) => {
  try { res.status(201).json(await Product.create(req.body)); }
  catch (e) { res.status(400).json({ error: e.message }); }
});
app.get("/api/products", async (req, res) => {
  res.json(await Product.find());
});
app.patch("/api/products/:id", async (req, res) => {
  try {
    const product = await Product.findByIdAndUpdate(
      req.params.id, { price: req.body.price },
      { new: true, runValidators: true }
    );
    if (!product) return res.status(404).json({ message: "Product not found" });
    res.json(product);
  } catch (e) { res.status(400).json({ error: e.message }); }
});
app.delete("/api/products/:id", async (req, res) => {
  const product = await Product.findByIdAndDelete(req.params.id);
  if (!product) return res.status(404).json({ message: "Product not found" });
  res.json({ message: "Product deleted" });
});
app.listen(5000, () => console.log("Server running on port 5000"));
 
// Backend setup:
// npm init -y
// npm install express mongoose cors
// node server.js
 
// FRONTEND: src/App.jsx
import { useEffect, useState } from "react";
 
const API = "http://localhost:5000/api/products";
 
export default function App() {
  const [products, setProducts] = useState([]);
  const [name, setName] = useState("");
  const [price, setPrice] = useState("");
  const [editPrices, setEditPrices] = useState({});
 
  const loadProducts = async () => {
    const response = await fetch(API);
    setProducts(await response.json());
  };
  useEffect(() => { loadProducts(); }, []);
 
  const addProduct = async (e) => {
    e.preventDefault();
    await fetch(API, {
      method: "POST", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name, price: Number(price) })
    });
    setName(""); setPrice(""); loadProducts();
  };
 
  const updatePrice = async (id) => {
    await fetch(`${API}/${id}`, {
      method: "PATCH", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ price: Number(editPrices[id]) })
    });
    loadProducts();
  };
 
  const deleteProduct = async (id) => {
    await fetch(`${API}/${id}`, { method: "DELETE" });
    loadProducts();
  };
 
  return (
    <main>
      <h1>Product Management</h1>
      <form onSubmit={addProduct}>
        <input placeholder="Product name" value={name}
          onChange={e => setName(e.target.value)} required />
        <input type="number" min="0" placeholder="Price" value={price}
          onChange={e => setPrice(e.target.value)} required />
        <button>Add Product</button>
      </form>
      {products.map(p => (
        <div key={p._id}>
          <b>{p.name}</b> — ₹{p.price}
          <input type="number" min="0" placeholder="New price"
            onChange={e => setEditPrices({ ...editPrices, [p._id]: e.target.value })} />
          <button onClick={() => updatePrice(p._id)}>Update Price</button>
          <button onClick={() => deleteProduct(p._id)}>Delete</button>
        </div>
      ))}
    </main>
  );
}
 
// Frontend setup:
// npm create vite@latest client -- --template react
// cd client
// npm install
// npm run dev
Output
Expected Output: Start MongoDB and the Express backend, then run the React frontend. Add a product using its name and price. The product list is loaded from MongoDB. Update Price changes the selected product's price, and Delete removes the selected product.

 Question
10. Deploy a basic MERN Stack application. Build the React frontend, deploy the Express backend, connect MongoDB Atlas, deploy the frontend and test the live URL.
Program
// 1. Prepare the Express backend for deployment.
// server/server.js
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const app = express();
 
app.use(cors({ origin: process.env.CLIENT_URL || "*" }));
app.use(express.json());
 
mongoose.connect(process.env.MONGODB_URI)
  .then(() => console.log("MongoDB Atlas connected"))
  .catch(err => console.error("MongoDB error:", err));
 
app.get("/", (req, res) => res.send("MERN API is running"));
app.get("/api/health", (req, res) => res.json({ status: "OK" }));
 
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server listening on ${PORT}`));
 
// server/package.json scripts:
// "scripts": { "start": "node server.js" }
 
// 2. In MongoDB Atlas:
// - Create a cluster and database user.
// - Allow the deployment host to connect in Network Access.
// - Copy the connection string and replace its password placeholder.
 
// 3. Configure backend environment variables on the hosting platform:
// MONGODB_URI = your MongoDB Atlas connection string
// CLIENT_URL = your deployed frontend URL
// PORT is generally provided by the hosting platform.
 
// 4. Build the React frontend:
// cd client
// npm install
// npm run build
 
// 5. Configure the frontend API URL using an environment variable.
// Example in client/.env.production:
// VITE_API_URL=https://your-backend-domain.example
 
// Use in React:
// const API_URL = import.meta.env.VITE_API_URL;
 
// 6. Deploy:
// - Push the project to a Git repository.
// - Create a backend web service from the server folder.
// - Set the backend start command to: npm start
// - Add MONGODB_URI and CLIENT_URL environment variables.
// - Create a static site from the client folder.
// - Set build command: npm run build
// - Set publish directory: dist
// - Set VITE_API_URL to the deployed backend URL.
// - Redeploy after setting environment variables.
 
// 7. Test:
// Open the deployed frontend URL.
// Open <backend-url>/api/health and verify { "status": "OK" }.
// Test the frontend-to-backend API request.
Output
Expected Output: The deployment is successful when the React frontend opens using its live URL, the backend health endpoint returns { status: "OK" }, and the backend connects to MongoDB Atlas. Test the deployed application by performing its available operations. Replace example domains and environment values with the actual values supplied by your hosting provider.


##################################################################################################################################################################################################################################################################################################################################################################################


Question
1. Create an Android application to demonstrate steps of Firebase Integration with Android app.
Program
// MainActivity.java
package com.example.firebaseintegration;
import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import com.google.firebase.FirebaseApp;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        TextView status = new TextView(this);
        status.setTextSize(22);
        setContentView(status);

        FirebaseApp.initializeApp(this);
        if (FirebaseApp.getApps(this).size() > 0) {
            status.setText("Firebase Integrated Successfully!");
        } else {
            status.setText("Firebase Initialization Failed");
        }
    }
}
Output
 
Question
2. Create an Android application to demonstrate steps of Firebase Integration with Android app.
Program
// MainActivity.java
package com.example.firebasecheck;
import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;
import com.google.firebase.FirebaseApp;
import com.google.firebase.database.FirebaseDatabase;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        TextView result = new TextView(this);
        result.setTextSize(20);
        setContentView(result);

        FirebaseApp.initializeApp(this);
        if (FirebaseApp.getApps(this).size() > 0) {
            result.setText("Firebase is ready to use.");
            // Example database reference (enable Realtime Database
            // and configure database rules in Firebase Console).
            FirebaseDatabase.getInstance().getReference("MAD/status")
                .setValue("Connected");
        } else {
            result.setText("Please check Firebase configuration.");
        }
    }
}
Output
 
Question
3. Create an Android application to show the notification/Alert window.
Program
// MainActivity.java
package com.example.alertdemo;
import android.os.Bundle;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.app.AlertDialog;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Button btn = new Button(this);
        btn.setText("Show Alert");
        setContentView(btn);

        btn.setOnClickListener(v -> new AlertDialog.Builder(this)
            .setTitle("MAD Application")
            .setMessage("This is an alert dialog message.")
            .setPositiveButton("OK", (dialog, which) -> dialog.dismiss())
            .setNegativeButton("Cancel", (dialog, which) -> dialog.dismiss())
            .show());
    }
}
Output
 
Question
4. Create an Android application to perform calling through intent.
Program
// MainActivity.java
package com.example.calldemo;
import android.os.Bundle;
import android.content.Intent;
import android.net.Uri;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        LinearLayout layout = new LinearLayout(this);
        layout.setOrientation(LinearLayout.VERTICAL);
        EditText phone = new EditText(this);
        phone.setHint("Enter phone number");
        Button call = new Button(this);
        call.setText("CALL");
        layout.addView(phone); layout.addView(call);
        setContentView(layout);

        call.setOnClickListener(v -> {
            String number = phone.getText().toString().trim();
            if (!number.isEmpty()) {
                Intent intent = new Intent(Intent.ACTION_DIAL,
                    Uri.parse("tel:" + number));
                startActivity(intent);
            } else {
                phone.setError("Enter phone number");
            }
        });
    }
}
Output
 
Question
5. Create an Android application to insert and display data using SQLite.
Program
// MainActivity.java
package com.example.sqlitedemo;
import android.os.Bundle;
import android.content.*;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.widget.*;

public class MainActivity extends Activity {
    DBHelper db;
    EditText name, email;
    TextView output;
    public void onCreate(Bundle b) {
        super.onCreate(b);
        db = new DBHelper(this);
        LinearLayout l = new LinearLayout(this);
        l.setOrientation(LinearLayout.VERTICAL);
        name = new EditText(this); name.setHint("Name");
        email = new EditText(this); email.setHint("Email");
        Button insert = new Button(this); insert.setText("Insert");
        Button show = new Button(this); show.setText("Display");
        output = new TextView(this);
        l.addView(name); l.addView(email); l.addView(insert);
        l.addView(show); l.addView(output); setContentView(l);

        insert.setOnClickListener(v -> {
            ContentValues cv = new ContentValues();
            cv.put("name", name.getText().toString());
            cv.put("email", email.getText().toString());
            db.getWritableDatabase().insert("student", null, cv);
            Toast.makeText(this, "Record Inserted", Toast.LENGTH_SHORT).show();
        });
        show.setOnClickListener(v -> {
            Cursor c = db.getReadableDatabase().rawQuery(
                "SELECT * FROM student", null);
            StringBuilder s = new StringBuilder();
            while (c.moveToNext())
                s.append(c.getInt(0)).append("  ")
                 .append(c.getString(1)).append("  ")
                 .append(c.getString(2)).append("\n");
            c.close(); output.setText(s.toString());
        });
    }
    class DBHelper extends SQLiteOpenHelper {
        DBHelper(Context c) { super(c, "CollegeDB", null, 1); }
        public void onCreate(SQLiteDatabase d) {
            d.execSQL("CREATE TABLE student(id INTEGER PRIMARY KEY AUTOINCREMENT,"
                + " name TEXT, email TEXT)");
        }
        public void onUpgrade(SQLiteDatabase d,int oldV,int newV) {
            d.execSQL("DROP TABLE IF EXISTS student"); onCreate(d);
        }
    }
}
Output
 
Question
6. Create an Android application to display popup menu.
Program
// MainActivity.java
package com.example.popupmenu;
import android.os.Bundle;
import android.view.*;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    TextView message;
    @Override
    protected void onCreate(Bundle b) {
        super.onCreate(b);
        LinearLayout layout = new LinearLayout(this);
        layout.setOrientation(LinearLayout.VERTICAL);
        Button btn = new Button(this); btn.setText("Show Popup Menu");
        message = new TextView(this);
        layout.addView(btn); layout.addView(message); setContentView(layout);

        btn.setOnClickListener(v -> {
            PopupMenu popup = new PopupMenu(this, btn);
            popup.getMenu().add("Edit");
            popup.getMenu().add("Share");
            popup.getMenu().add("Delete");
            popup.setOnMenuItemClickListener(item -> {
                message.setText(item.getTitle() + " selected");
                return true;
            });
            popup.show();
        });
    }
}
Output
 
Question
7. Create an Android application to accept data using custom adapter and display it.
Program
// MainActivity.java
package com.example.customadapter;
import android.os.Bundle;
import android.widget.*;
import android.view.*;
import androidx.appcompat.app.AppCompatActivity;
import java.util.*;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle b) {
        super.onCreate(b);
        ListView list = new ListView(this);
        ArrayList<String> students = new ArrayList<>();
        students.add("Asha - MCA");
        students.add("Rahul - MCA");
        students.add("Neha - MCA");
        StudentAdapter adapter = new StudentAdapter(students);
        list.setAdapter(adapter);
        setContentView(list);
    }
    class StudentAdapter extends BaseAdapter {
        ArrayList<String> data;
        StudentAdapter(ArrayList<String> d) { data = d; }
        public int getCount() { return data.size(); }
        public Object getItem(int p) { return data.get(p); }
        public long getItemId(int p) { return p; }
        public View getView(int p, View v, android.view.ViewGroup parent) {
            TextView row = new TextView(MainActivity.this);
            row.setText(data.get(p)); row.setTextSize(22);
            row.setPadding(24, 24, 24, 24);
            return row;
        }
    }
}
Output
 
Question
8. Create a React application to accept the details of the user and display them.
Program
// App.js (React)
import React, { useState } from "react";

export default function App() {
  const [form, setForm] = useState({ name: "", email: "", course: "" });
  const [submitted, setSubmitted] = useState(null);

  const handleChange = (e) =>
    setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = (e) => {
    e.preventDefault();
    setSubmitted(form);
  };

  return (
    <div>
      <h2>User Details</h2>
      <form onSubmit={handleSubmit}>
        <input name="name" placeholder="Name" value={form.name}
          onChange={handleChange} required />
        <input name="email" type="email" placeholder="Email"
          value={form.email} onChange={handleChange} required />
        <input name="course" placeholder="Course" value={form.course}
          onChange={handleChange} required />
        <button type="submit">Submit</button>
      </form>
      {submitted && <div>
        <h3>Entered Details</h3>
        <p>Name: {submitted.name}</p>
        <p>Email: {submitted.email}</p>
        <p>Course: {submitted.course}</p>
      </div>}
    </div>
  );
}
Output
 
Question
9. Create a React mobile application to print addition of two numbers.
Program
// App.js (React Native)
import React, { useState } from "react";
import { View, Text, TextInput, Button, StyleSheet } from "react-native";

export default function App() {
  const [a, setA] = useState("");
  const [b, setB] = useState("");
  const [sum, setSum] = useState(null);

  const addNumbers = () => {
    const x = Number(a), y = Number(b);
    if (a.trim() !== "" && b.trim() !== "")
      setSum(x + y);
    else
      setSum("Please enter both numbers");
  };

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Addition of Two Numbers</Text>
      <TextInput style={styles.input} placeholder="First number"
        keyboardType="numeric" value={a} onChangeText={setA} />
      <TextInput style={styles.input} placeholder="Second number"
        keyboardType="numeric" value={b} onChangeText={setB} />
      <Button title="Add Numbers" onPress={addNumbers} />
      <Text style={styles.result}>Result: {sum}</Text>
    </View>
  );
}
const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", padding: 24 },
  heading: { fontSize: 22, marginBottom: 20 },
  input: { borderWidth: 1, padding: 12, marginBottom: 12 },
  result: { fontSize: 20, marginTop: 20 }
});
Output
 
Question
10. Create a simple Flutter application of Hello World.
Program
// lib/main.dart (Flutter)
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Hello World App')),
        body: const Center(
          child: Text(
            'Hello World!',
            style: TextStyle(fontSize: 28),
          ),
        ),
      ),
    );
  }
}
Output
 
