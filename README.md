mkdir ticket-booking
cd ticket-booking
npm init -y
npm install express
const express = require('express');
const app = express();
const PORT = 3000;

app.use(express.json());

// In-memory seat storage
// Each seat has { id, status, lockBy, lockTime }
let seats = [];
const TOTAL_SEATS = 10; // Example: 10 seats

for (let i = 1; i <= TOTAL_SEATS; i++) {
  seats.push({ id: i, status: 'available', lockBy: null, lockTime: null });
}

// Lock expiry time in milliseconds (1 minute)
const LOCK_EXPIRY = 60 * 1000;

// Utility to clean expired locks
function cleanExpiredLocks() {
  const now = Date.now();
  seats.forEach(seat => {
    if (seat.status === 'locked' && now - seat.lockTime > LOCK_EXPIRY) {
      seat.status = 'available';
      seat.lockBy = null;
      seat.lockTime = null;
      console.log(`Lock expired for seat ${seat.id}`);
    }
  });
}

// GET /seats - View all seats
app.get('/seats', (req, res) => {
  cleanExpiredLocks();
  res.json(seats);
});

// POST /lock/:seatId - Lock a seat
app.post('/lock/:seatId', (req, res) => {
  const seatId = parseInt(req.params.seatId);
  const user = req.body.user;

  const seat = seats.find(s => s.id === seatId);
  if (!seat) return res.status(404).json({ message: 'Seat not found' });

  cleanExpiredLocks();

  if (seat.status === 'booked') {
    return res.status(400).json({ message: 'Seat already booked' });
  }
  if (seat.status === 'locked') {
    return res.status(400).json({ message: `Seat is locked by ${seat.lockBy}` });
  }

  seat.status = 'locked';
  seat.lockBy = user;
  seat.lockTime = Date.now();

  res.json({ message: `Seat ${seat.id} locked successfully by ${user}` });
});

// POST /confirm/:seatId - Confirm a booking
app.post('/confirm/:seatId', (req, res) => {
  const seatId = parseInt(req.params.seatId);
  const user = req.body.user;

  const seat = seats.find(s => s.id === seatId);
  if (!seat) return res.status(404).json({ message: 'Seat not found' });

  cleanExpiredLocks();

  if (seat.status !== 'locked' || seat.lockBy !== user) {
    return res.status(400).json({ message: 'Seat not locked by you or lock expired' });
  }

  seat.status = 'booked';
  seat.lockBy = null;
  seat.lockTime = null;

  res.json({ message: `Seat ${seat.id} booked successfully by ${user}` });
});

// POST /unlock/:seatId - Optional: release a lock manually
app.post('/unlock/:seatId', (req, res) => {
  const seatId = parseInt(req.params.seatId);
  const user = req.body.user;

  const seat = seats.find(s => s.id === seatId);
  if (!seat) return res.status(404).json({ message: 'Seat not found' });

  if (seat.status === 'locked' && seat.lockBy === user) {
    seat.status = 'available';
    seat.lockBy = null;
    seat.lockTime = null;
    return res.json({ message: `Lock released for seat ${seat.id}` });
  }

  res.status(400).json({ message: 'You do not have a lock on this seat' });
});

// Start server
app.listen(PORT, () => {
  console.log(`Ticket booking server running at http://localhost:${PORT}`);
});
[
  {"id":1,"status":"available","lockBy":null,"lockTime":null},
  {"id":2,"status":"available","lockBy":null,"lockTime":null},
  ...
]
