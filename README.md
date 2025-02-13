STEP 1:
--Employees table 
CREATE TABLE employees (
   id INTEGER PRIMARY KEY AUTOINCREMENT,
   name TEXT NOT NULL,
   email TEXT UNIQUE NOT NULL CHECK(email LIKE '%@%.%'),
   phone TEXT NOT NULL CHECK(length(phone) >= 10),
   date_of_birth DATE NOT NULL CHECK(julianday('now') - julianday(date_of_birth)
   job_title TEXT NOT NULL,
   department TEXT NOT NULL,
   salary REAL NOT NULL CHECK(salary >= 300),
   start_date DATE NOT NULL,
   end_date DATE NULL,
   photo TEXT NULL,
   documents TEXT NULL
   );
   
--Timesheets Table
CREATE TABLE timesheets (
   id INTEGER PRIMARY KEY AUTOINCREMENT,
   employee_id INTEGER NOT NULL,
   start_time DATETIME NOT NULL,
   end_time DATETIME NOT NULL CHECK(end_time > start_time),
   summary TEXT NULL, 
   FOREIGN KEY (employee_id) REFERENCES employees(id) ON DELETE CASCADE
   );
   
STEP 2:

const db = require("../app/db");

async function seed() {
await db.run('
  INSERT INTO employees 
  (name,email,phone,date_of_birth,job_title, department, salary, start_date)
  VALUES 
  ('Alice Johnson', 'alice@example.com', '1234567890', '1990-05-10', 'Software Engineer', 'IT', 80000, '2022-01-01'), 
  );

await db.run('
INSERT INTO timesheets (epmloyee_id, start_time, end_time, summary)
VALUES 
(1, '2025-01-01 09:00:00', 2025-10-10 17:00:00', 'Completed backend API updates')
);
}
seed();

npm run setup_db
npm run seed

Step 3: 

import { Link, useLoaderData } from "react-router-dom";
import db from "../db";

export async function loader() {
  const employees = await db.all("SELECT id, name, job_title, department, salary FROM employees");
  return employees;
}

export default function EmployeeList() {
  const employees = useLoaderData();

  return (
    <div>
      <h1>Employees</h1>
      <Link to="/employees/new">➕ Add Employee</Link>
      <Link to="/timesheets">📄 View Timesheets</Link>
      
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Job Title</th>
            <th>Department</th>
            <th>Salary</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          {employees.map((emp) => (
            <tr key={emp.id}>
              <td>{emp.name}</td>
              <td>{emp.job_title}</td>
              <td>{emp.department}</td>
              <td>${emp.salary}</td>
              <td><Link to={`/employees/${emp.id}`}>View</Link></td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

Step 4:

import { Form, useLoaderData } from "react-router-dom";

export async function loader({ params }) {
  const employee = params.employeeId
    ? await db.get("SELECT * FROM employees WHERE id = ?", [params.employeeId])
    : null;
  return employee;
}

export async function action({ request }) {
  const formData = await request.formData();
  const employee = Object.fromEntries(formData);

  if (employee.id) {
    await db.run(`
      UPDATE employees SET name=?, email=?, phone=?, date_of_birth=?, job_title=?, department=?, salary=?, start_date=?, end_date=?
      WHERE id=?
    `, [employee.name, employee.email, employee.phone, employee.date_of_birth, employee.job_title, employee.department, employee.salary, employee.start_date, employee.end_date, employee.id]);
  } else {
    await db.run(`
      INSERT INTO employees (name, email, phone, date_of_birth, job_title, department, salary, start_date)
      VALUES (?, ?, ?, ?, ?, ?, ?, ?)
    `, [employee.name, employee.email, employee.phone, employee.date_of_birth, employee.job_title, employee.department, employee.salary, employee.start_date]);
  }
  return null;
}

export default function EmployeeForm() {
  const employee = useLoaderData();

  return (
    <Form method="post">
      <input type="hidden" name="id" value={employee?.id} />
      <label>Name: <input name="name" defaultValue={employee?.name} required /></label>
      <label>Email: <input type="email" name="email" defaultValue={employee?.email} required /></label>
      <label>Phone: <input type="tel" name="phone" defaultValue={employee?.phone} required /></label>
      <label>Job Title: <input name="job_title" defaultValue={employee?.job_title} required /></label>
      <label>Department: <input name="department" defaultValue={employee?.department} required /></label>
      <label>Salary: <input type="number" name="salary" defaultValue={employee?.salary} required /></label>
      <button type="submit">Save</button>
    </Form>
  );
}

npm run setup_db
npm run seed
npm run dev



   
 
