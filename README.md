<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Screen Configuration System</title>
    <style>
        .seat 
        {
            width: 50px;
            height: 50px;
            margin: 5px;
            background-color: green;
            display: inline-block;
            cursor: pointer;
        }
        .locked 
        {
            background-color: yellow;
        }
        .booked 
        {
            background-color: red;
            cursor: not-allowed;
        }
        .screen-selection 
        {
            margin-bottom: 20px;
        }
    </style>
</head>
<body>
    <h1>Screen Configuration System</h1>
    <div class="screen-selection">
        <label for="screens">Select Screen: </label>
        <select id="screens">
            <option value="1">Screen 1</option>
            <option value="2">Screen 2</option>
            <option value="3">Screen 3</option>
        </select>
    </div>
    <div id="seats"></div>
    <button id="confirmBooking">Confirm Booking</button>
    <script>
        const screens = {
            1: { capacity: 60, seats: Array.from({ length: 60 }, (_, i) => ({ id: i + 1, status: 'available' })) },
            2: { capacity: 50, seats: Array.from({ length: 50 }, (_, i) => ({ id: i + 1, status: 'available' })) },
            3: { capacity: 40, seats: Array.from({ length: 40 }, (_, i) => ({ id: i + 1, status: 'available' })) }
        };

        const seatContainer = document.getElementById('seats');
        const screenSelect = document.getElementById('screens');

        function renderSeats(screenId) 
       {
            seatContainer.innerHTML = '';
            const seats = screens[screenId].seats;
            seats.forEach(seat => 
           {
                const seatElement = document.createElement('div');
                seatElement.classList.add('seat');
                seatElement.dataset.id = seat.id;
                seatElement.dataset.screen = screenId;
                seatContainer.appendChild(seatElement);

                seatElement.addEventListener('click', () => 
                    {
                    if (seat.status === 'available') 
                    {
                        seat.status = 'locked';
                        seatElement.classList.add('locked');
                        lockSeat(screenId, seat.id);
                    } else if (seat.status === 'locked') 
                    {
                        seat.status = 'available';
                        seatElement.classList.remove('locked');
                        releaseSeat(screenId, seat.id);
                    }
                });
            });
        }

        function lockSeat(screenId, seatId) 
        {
            fetch(`/lock-seat/${screenId}/${seatId}`, { method: 'POST' })
                .then(response => response.json())
                .then(data => 
                {
                    if (!data.success) 
                    {
                        alert('Failed to lock seat.');
                        document.querySelector(`.seat[data-id="${seatId}"]`).classList.remove('locked');
                        screens[screenId].seats.find(seat => seat.id === seatId).status = 'available';
                    }
                });
        }

        function releaseSeat(screenId, seatId) 
        {
            fetch(`/release-seat/${screenId}/${seatId}`, { method: 'POST' });
        }

        document.getElementById('confirmBooking').addEventListener('click', () => 
            {
            const screenId = screenSelect.value;
            const lockedSeats = screens[screenId].seats.filter(seat => seat.status === 'locked');
            if (lockedSeats.length === 0) 
            {
                alert('No seats selected.');
                return;
            }

            const seatIds = lockedSeats.map(seat => seat.id);

            fetch('/confirm-booking', 
                {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ screenId, seatIds })
            })
                .then(response => response.json())
                .then(data => 
                 {
                    if (data.success) 
                    {
                        alert('Booking confirmed.');
                        lockedSeats.forEach(seat => 
                        {
                            seat.status = 'booked';
                            document.querySelector(`.seat[data-id="${seat.id}"][data-screen="${screenId}"]`).classList.add('booked');
                            document.querySelector(`.seat[data-id="${seat.id}"][data-screen="${screenId}"]`).classList.remove('locked');
                        });
                    } else 
                    {
                        alert('Failed to confirm booking.');
                    }
                });
        });

        screenSelect.addEventListener('change', () => 
        {
            renderSeats(screenSelect.value);
        });

        renderSeats(screenSelect.value);
    </script>
</body>
</html>
