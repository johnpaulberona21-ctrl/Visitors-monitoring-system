<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Visitors Monitoring System</title>

    <style>
        * {
            box-sizing: border-box;
            font-family: Arial, sans-serif;
        }

        body {
            margin: 0;
            background: #f4f6f8;
            color: #222;
        }

        header {
            background: #1f4e79;
            color: white;
            padding: 25px;
            text-align: center;
        }

        header h1 {
            margin: 0;
        }

        .container {
            width: 90%;
            max-width: 1100px;
            margin: 25px auto;
        }

        .card {
            background: white;
            padding: 20px;
            margin-bottom: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }

        h2 {
            margin-top: 0;
            color: #1f4e79;
        }

        .form-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 15px;
        }

        input, select {
            width: 100%;
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 6px;
        }

        button {
            border: none;
            padding: 12px 18px;
            border-radius: 6px;
            cursor: pointer;
            color: white;
            background: #1f4e79;
            margin-top: 15px;
        }

        button:hover {
            opacity: 0.9;
        }

        .delete-btn {
            background: #d9534f;
            margin: 0;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
        }

        th, td {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: center;
        }

        th {
            background: #1f4e79;
            color: white;
        }

        .stats {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
        }

        .stat-box {
            padding: 20px;
            text-align: center;
            border-radius: 10px;
            background: #e8f1f8;
        }

        .stat-box h3 {
            margin: 0;
            font-size: 30px;
            color: #1f4e79;
        }

        @media (max-width: 700px) {
            .form-grid,
            .stats {
                grid-template-columns: 1fr;
            }

            table {
                font-size: 12px;
            }
        }
    </style>
</head>

<body>

<header>
    <h1>Visitors Monitoring System</h1>
    <p>Visitor Registration and Monitoring</p>
</header>

<div class="container">

    <!-- Statistics -->
    <div class="card">
        <h2>Dashboard</h2>

        <div class="stats">
            <div class="stat-box">
                <h3 id="totalVisitors">0</h3>
                <p>Total Visitors</p>
            </div>

            <div class="stat-box">
                <h3 id="insideVisitors">0</h3>
                <p>Currently Inside</p>
            </div>

            <div class="stat-box">
                <h3 id="exitedVisitors">0</h3>
                <p>Exited Visitors</p>
            </div>
        </div>
    </div>

    <!-- Registration Form -->
    <div class="card">
        <h2>Visitor Registration</h2>

        <form id="visitorForm">

            <div class="form-grid">

                <div>
                    <label>Full Name</label>
                    <input type="text" id="name" required>
                </div>

                <div>
                    <label>Contact Number</label>
                    <input type="text" id="contact" required>
                </div>

                <div>
                    <label>Purpose of Visit</label>
                    <input type="text" id="purpose" required>
                </div>

                <div>
                    <label>Person to Visit</label>
                    <input type="text" id="person" required>
                </div>

            </div>

            <button type="submit">Register Visitor</button>

        </form>
    </div>

    <!-- Visitor Records -->
    <div class="card">
        <h2>Visitor Records</h2>

        <table>
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Contact</th>
                    <th>Purpose</th>
                    <th>Person to Visit</th>
                    <th>Time In</th>
                    <th>Status</th>
                    <th>Action</th>
                </tr>
            </thead>

            <tbody id="visitorTable">
            </tbody>
        </table>
    </div>

</div>

<script>

    let visitors = JSON.parse(localStorage.getItem("visitors")) || [];

    const form = document.getElementById("visitorForm");
    const table = document.getElementById("visitorTable");

    form.addEventListener("submit", function(event) {

        event.preventDefault();

        const visitor = {
            id: Date.now(),
            name: document.getElementById("name").value,
            contact: document.getElementById("contact").value,
            purpose: document.getElementById("purpose").value,
            person: document.getElementById("person").value,
            timeIn: new Date().toLocaleString(),
            status: "Inside"
        };

        visitors.push(visitor);

        saveData();

        form.reset();

        displayVisitors();

        alert("Visitor successfully registered!");

    });

    function displayVisitors() {

        table.innerHTML = "";

        visitors.forEach(function(visitor) {

            const row = document.createElement("tr");

            row.innerHTML = `
                <td>${visitor.name}</td>
                <td>${visitor.contact}</td>
                <td>${visitor.purpose}</td>
                <td>${visitor.person}</td>
                <td>${visitor.timeIn}</td>

                <td>
                    ${visitor.status}
                </td>

                <td>
                    ${
                        visitor.status === "Inside"
                        ?
                        `<button onclick="checkOut(${visitor.id})">
                            Check Out
                         </button>`
                        :
                        "Completed"
                    }

                    <button
                        class="delete-btn"
                        onclick="deleteVisitor(${visitor.id})">
                        Delete
                    </button>
                </td>
            `;

            table.appendChild(row);

        });

        updateDashboard();
    }

    function checkOut(id) {

        const visitor = visitors.find(function(visitor) {
            return visitor.id === id;
        });

        if (visitor) {
            visitor.status = "Exited";
            visitor.timeOut = new Date().toLocaleString();
        }

        saveData();
        displayVisitors();

    }

    function deleteVisitor(id) {

        if (confirm("Delete this visitor record?")) {

            visitors = visitors.filter(function(visitor) {
                return visitor.id !== id;
            });

            saveData();
            displayVisitors();
        }
    }

    function updateDashboard() {

        const total = visitors.length;

        const inside = visitors.filter(function(visitor) {
            return visitor.status === "Inside";
        }).length;

        const exited = visitors.filter(function(visitor) {
            return visitor.status === "Exited";
        }).length;

        document.getElementById("totalVisitors").textContent = total;
        document.getElementById("insideVisitors").textContent = inside;
        document.getElementById("exitedVisitors").textContent = exited;
    }

    function saveData() {

        localStorage.setItem(
            "visitors",
            JSON.stringify(visitors)
        );

    }

    displayVisitors();

</script>

</body>
</html>
