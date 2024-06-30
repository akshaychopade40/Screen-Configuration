<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Screen Configuration System</title>
    <style>
        body 
        {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f7f7f7;
            padding: 20px;
        }
        .container 
        {
            text-align: center;
            margin-top: 20px;
        }
        .screen-selector 
        {
            margin-bottom: 20px;
        }
        .seat 
        {
            width: 50px;
            height: 50px;
            margin: 5px;
            display: inline-block;
            background-color: #ccc;
            cursor: pointer;
        }
        .seat.available 
        {
            background-color: #28a745;
        }
        .seat.locked 
        {
            background-color: #ffc107;
        }
        .seat.booked 
         {
            background-color: #dc3545;
            cursor: not-allowed;
        }
        .seat.selected 
        {
            background-color: #007bff;
        }
        #message 
        {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="screen-selector">
            <label for="screen">Select Screen:</label>
            <select id="screen" onchange="selectScreen()">
                <option value="screen1">Screen 1 (60 seats)</option>
                <option value="screen2">Screen 2 (50 seats)</option>
                <option value="screen3">Screen 3 (40 seats)</option>
            </select>
        </div>
        <div id="seats"></div>
        <div id="message"></div>
    </div>

    <script>
        const screens = 
        {
            screen1: 60,
            screen2: 50,
            screen3: 40
        };
        let selectedScreen = 'screen1';
        let seats = [];

        function createSeats(screen) 
      {
            const seatsContainer = document.getElementById('seats');
            seatsContainer.innerHTML = '';
            seats = [];
            for (let i = 0; i < screens[screen]; i++) 
            {
                const seat = document.createElement('div');
                seat.className = 'seat available';
                seat.dataset.index = i;
                seat.addEventListener('click', () => selectSeat(i));
                seatsContainer.appendChild(seat);
                seats.push({
                    element: seat,
                    status: 'available',
                    lockTimer: null
                });
            }
        }

        function selectScreen() 
        {
            selectedScreen = document.getElementById('screen').value;
            createSeats(selectedScreen);
        }

        function selectSeat(index) 
        {
            const seat = seats[index];
            if (seat.status === 'available') 
            {
                lockSeat(index);
            } else if (seat.status === 'selected') 
            {
                releaseSeat(index);
            }
        }

        function lockSeat(index) 
        {
            const seat = seats[index];
            seat.status = 'locked';
            seat.element.className = 'seat locked';
            seat.lockTimer = setTimeout(() => releaseSeat(index), 30000); 
            displayMessage(`Seat ${index + 1} locked. Complete booking within 15 seconds.`);
        }

        function releaseSeat(index) 
        {
            const seat = seats[index];
            if (seat.status === 'locked') 
            {
                seat.status = 'available';
                seat.element.className = 'seat available';
                clearTimeout(seat.lockTimer);
                seat.lockTimer = null;
                displayMessage(`Seat ${index + 1} is available again.`);
            }
        }

        function displayMessage(message) 
        {
            const messageDiv = document.getElementById('message');
            messageDiv.innerText = message;
        }

        
        createSeats(selectedScreen);
    </script>
</body>
</html>
