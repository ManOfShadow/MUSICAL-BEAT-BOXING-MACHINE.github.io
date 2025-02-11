<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Piezometer Data</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            background-color: #f4f4f9; 
            text-align: center; 
            padding: 20px; 
        }
        h2 { color: #333; }
        button { 
            padding: 10px 20px; 
            font-size: 16px; 
            margin: 20px 0; 
            cursor: pointer; 
            background-color: #4CAF50; 
            color: white; 
            border: none; 
            border-radius: 5px;
            transition: background-color 0.3s;
        }
        button:hover { background-color: #45a049; }
        .container { 
            display: flex; 
            justify-content: space-around; 
            flex-wrap: wrap; 
            gap: 20px;
        }
        .sensor-box {
            background-color: white; 
            border-radius: 10px; 
            padding: 20px; 
            width: 300px; 
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); 
            margin-top: 20px;
        }
        .sensor-box h3 {
            color: #444;
        }
        canvas {
            max-width: 100%;
            height: 250px;
        }
        #liveData { 
            font-size: 18px; 
            font-weight: bold; 
            color: #333; 
        }
        ul {
            text-align: left;
            list-style-type: none;
            padding: 0;
            margin: 10px 0;
            font-size: 14px;
            color: #555;
        }
        li {
            padding: 5px 0;
            border-bottom: 1px solid #eee;
        }
        li:last-child { border-bottom: none; }
    </style>
</head>
<body>

    <h2>Piezometer Sensor Data</h2>
    <button onclick="connectBluetooth()">🔵 Connect via Bluetooth</button>

    <div class="container">
        <!-- Piezometer 1 Section -->
        <div class="sensor-box">
            <h3>Piezometer 1</h3>
            <p id="liveData1">Waiting for data...</p>
            <ul id="pastValues1"></ul>
            <canvas id="dataChart1"></canvas>
        </div>

        <!-- Piezometer 2 Section -->
        <div class="sensor-box">
            <h3>Piezometer 2</h3>
            <p id="liveData2">Waiting for data...</p>
            <ul id="pastValues2"></ul>
            <canvas id="dataChart2"></canvas>
        </div>

        <!-- Piezometer 3 Section -->
        <div class="sensor-box">
            <h3>Piezometer 3</h3>
            <p id="liveData3">Waiting for data...</p>
            <ul id="pastValues3"></ul>
            <canvas id="dataChart3"></canvas>
        </div>

        <!-- Piezometer 4 Section -->
        <div class="sensor-box">
            <h3>Piezometer 4</h3>
            <p id="liveData4">Waiting for data...</p>
            <ul id="pastValues4"></ul>
            <canvas id="dataChart4"></canvas>
        </div>
    </div>

    <script>
        let sensorValues1 = [];
        let sensorValues2 = [];
        let sensorValues3 = [];
        let sensorValues4 = [];
        let timeLabels = [];

        const ctx1 = document.getElementById('dataChart1').getContext('2d');
        const ctx2 = document.getElementById('dataChart2').getContext('2d');
        const ctx3 = document.getElementById('dataChart3').getContext('2d');
        const ctx4 = document.getElementById('dataChart4').getContext('2d');

        const chart1 = new Chart(ctx1, {
            type: 'line',
            data: {
                labels: timeLabels,
                datasets: [{
                    label: 'Piezometer 1',
                    borderColor: 'red',
                    data: sensorValues1,
                    fill: false
                }]
            },
            options: { responsive: true }
        });

        const chart2 = new Chart(ctx2, {
            type: 'line',
            data: {
                labels: timeLabels,
                datasets: [{
                    label: 'Piezometer 2',
                    borderColor: 'green',
                    data: sensorValues2,
                    fill: false
                }]
            },
            options: { responsive: true }
        });

        const chart3 = new Chart(ctx3, {
            type: 'line',
            data: {
                labels: timeLabels,
                datasets: [{
                    label: 'Piezometer 3',
                    borderColor: 'blue',
                    data: sensorValues3,
                    fill: false
                }]
            },
            options: { responsive: true }
        });

        const chart4 = new Chart(ctx4, {
            type: 'line',
            data: {
                labels: timeLabels,
                datasets: [{
                    label: 'Piezometer 4',
                    borderColor: 'orange',
                    data: sensorValues4,
                    fill: false
                }]
            },
            options: { responsive: true }
        });

        async function connectBluetooth() {
            try {
                const device = await navigator.bluetooth.requestDevice({
                    acceptAllDevices: true,
                    optionalServices: ['12345678-1234-1234-1234-123456789abc']
                });

                const server = await device.gatt.connect();
                const service = await server.getPrimaryService('12345678-1234-1234-1234-123456789abc');
                const characteristic = await service.getCharacteristic('87654321-4321-4321-4321-123456789abc');

                characteristic.startNotifications();
                characteristic.addEventListener('characteristicvaluechanged', event => {
                    const value = new TextDecoder().decode(event.target.value);
                    updateUI(value);
                });

                alert("Connected!");
            } catch (error) {
                console.error(error);
                alert("Bluetooth Connection Failed");
            }
        }

        function updateUI(data) {
            let jsonData = JSON.parse(data);
            let currentTime = new Date().toLocaleTimeString();

            // Update live data for each piezometer
            document.getElementById("liveData1").innerText = `P1: ${jsonData.piezometer_1}`;
            document.getElementById("liveData2").innerText = `P2: ${jsonData.piezometer_2}`;
            document.getElementById("liveData3").innerText = `P3: ${jsonData.piezometer_3}`;
            document.getElementById("liveData4").innerText = `P4: ${jsonData.piezometer_4}`;

            // Add to past readings for each piezometer
            let listItem1 = document.createElement("li");
            listItem1.innerText = `${currentTime} - P1: ${jsonData.piezometer_1}`;
            document.getElementById("pastValues1").prepend(listItem1);

            let listItem2 = document.createElement("li");
            listItem2.innerText = `${currentTime} - P2: ${jsonData.piezometer_2}`;
            document.getElementById("pastValues2").prepend(listItem2);

            let listItem3 = document.createElement("li");
            listItem3.innerText = `${currentTime} - P3: ${jsonData.piezometer_3}`;
            document.getElementById("pastValues3").prepend(listItem3);

            let listItem4 = document.createElement("li");
            listItem4.innerText = `${currentTime} - P4: ${jsonData.piezometer_4}`;
            document.getElementById("pastValues4").prepend(listItem4);

            // Update the graphs for each piezometer
            if (sensorValues1.length > 20) { sensorValues1.shift(); }
            if (sensorValues2.length > 20) { sensorValues2.shift(); }
            if (sensorValues3.length > 20) { sensorValues3.shift(); }
            if (sensorValues4.length > 20) { sensorValues4.shift(); }
            if (timeLabels.length > 20) { timeLabels.shift(); }

            sensorValues1.push(jsonData.piezometer_1);
            sensorValues2.push(jsonData.piezometer_2);
            sensorValues3.push(jsonData.piezometer_3);
            sensorValues4.push(jsonData.piezometer_4);
            timeLabels.push(currentTime);

            chart1.update();
            chart2.update();
            chart3.update();
            chart4.update();
        }
    </script>
</body>
</html>
