// --- Synthesized Audio Engine ---
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

function playSynthSound(type) {
  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }
  
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.connect(gain);
  gain.connect(audioCtx.destination);

  const now = audioCtx.currentTime;

  if (type === 'click') {
    osc.type = 'sine';
    osc.frequency.setValueAtTime(300, now);
    osc.frequency.exponentialRampToValueAtTime(600, now + 0.04);
    gain.gain.setValueAtTime(0.08, now);
    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.04);
    osc.start(now);
    osc.stop(now + 0.04);
  } else if (type === 'success') {
    osc.type = 'triangle';
    osc.frequency.setValueAtTime(523.25, now);
    osc.frequency.setValueAtTime(659.25, now + 0.08);
    gain.gain.setValueAtTime(0.1, now);
    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.22);
    osc.start(now);
    osc.stop(now + 0.22);
  }
}

// --- Multi-User State & Persistence ---
let currentUser = localStorage.getItem('activeUser') || '';

function getClients() {
  if (!currentUser) return [];
  return JSON.parse(localStorage.getItem(`clients_${currentUser}`)) || [];
}

function saveClients(clients) {
  if (!currentUser) return;
  localStorage.setItem(`clients_${currentUser}`, JSON.stringify(clients));
}

// User Sign In Elements
const loginOverlay = document.getElementById('loginOverlay');
const loginForm = document.getElementById('loginForm');
const currentUserDisplay = document.getElementById('currentUserDisplay');
const switchUserBtn = document.getElementById('switchUserBtn');

function checkUserAuth() {
  if (!currentUser) {
    loginOverlay.style.display = 'flex';
  } else {
    loginOverlay.style.display = 'none';
    currentUserDisplay.textContent = currentUser;
    renderApp();
  }
}

loginForm.addEventListener('submit', (e) => {
  e.preventDefault();
  const inputVal = document.getElementById('usernameInput').value.trim();
  if (inputVal) {
    currentUser = inputVal;
    localStorage.setItem('activeUser', currentUser);
    playSynthSound('success');
    checkUserAuth();
  }
});

switchUserBtn.addEventListener('click', () => {
  playSynthSound('click');
  currentUser = '';
  localStorage.removeItem('activeUser');
  checkUserAuth();
});

// Modal Controls
const modal = document.getElementById('clientModal');
const openModalBtn = document.getElementById('openModalBtn');
const closeModalBtn = document.getElementById('closeModalBtn');
const cancelModalBtn = document.getElementById('cancelModalBtn');
const clientForm = document.getElementById('clientForm');

function openModal() {
  playSynthSound('click');
  modal.style.display = 'flex';
}

function closeModal() {
  playSynthSound('click');
  modal.style.display = 'none';
  clientForm.reset();
}

openModalBtn.addEventListener('click', openModal);
closeModalBtn.addEventListener('click', closeModal);
cancelModalBtn.addEventListener('click', closeModal);

// --- Appointment Reminder Engine ---
function checkUpcomingNotifications(clients) {
  const now = new Date();
  
  const upcomingList = clients
    .filter(c => c.appointment && new Date(c.appointment) > now)
    .sort((a, b) => new Date(a.appointment) - new Date(b.appointment));

  const banner = document.getElementById('notificationBanner');
  const text = document.getElementById('notificationText');

  if (upcomingList.length > 0) {
    const nextClient = upcomingList[0];
    const apptDate = new Date(nextClient.appointment);
    
    const dayName = apptDate.toLocaleDateString([], { weekday: 'long' });
    const monthName = apptDate.toLocaleDateString([], { month: 'long' });
    const dayNum = apptDate.getDate();
    
    let ordinal = 'th';
    if (dayNum === 1 || dayNum === 21 || dayNum === 31) ordinal = 'st';
    else if (dayNum === 2 || dayNum === 22) ordinal = 'nd';
    else if (dayNum === 3 || dayNum === 23) ordinal = 'rd';

    const formattedTime = apptDate.toLocaleTimeString([], { hour: 'numeric', minute: '2-digit' });
    
    const reminderMsg = `Don't forget! ${nextClient.name}'s appointment is scheduled for ${dayName}, ${monthName} ${dayNum}${ordinal} at ${formattedTime}`;

    text.textContent = reminderMsg;
    banner.style.display = 'flex';

    if ("Notification" in window && Notification.permission === "granted") {
      new Notification("ClientIndex Reminder", { body: reminderMsg });
    } else if ("Notification" in window && Notification.permission !== "denied") {
      Notification.requestPermission();
    }
  } else {
    banner.style.display = 'none';
  }
}

function dismissNotification() {
  playSynthSound('click');
  document.getElementById('notificationBanner').style.display = 'none';
}

function renderApp() {
  const clients = getClients();
  const tableBody = document.getElementById('clientTableBody');
  tableBody.innerHTML = '';

  let totalRevenue = 0;
  let upcomingAppointments = 0;

  if (clients.length === 0) {
    tableBody.innerHTML = `
      <tr>
        <td colspan="6" style="text-align: center; color: #718096; padding: 24px;">
          Your directory is currently empty. Click "+ New Client" above to get started!
        </td>
      </tr>`;
  } else {
    clients.forEach((client, index) => {
      const paidAmount = parseFloat(client.paid) || 0;
      totalRevenue += paidAmount;

      let formattedDate = 'None scheduled';
      if (client.appointment) {
        if (new Date(client.appointment) > new Date()) {
          upcomingAppointments++;
        }
        formattedDate = new Date(client.appointment).toLocaleString([], {
          dateStyle: 'medium',
          timeStyle: 'short'
        });
      }

      const row = document.createElement('tr');
      row.innerHTML = `
        <td><strong>${client.name}</strong></td>
        <td>${client.phone ? '📞 ' + client.phone : 'N/A'}</td>
        <td><span class="status-badge ${client.status.toLowerCase()}">${client.status}</span></td>
        <td>$${paidAmount.toFixed(2)}</td>
        <td>${formattedDate}</td>
        <td>
          <button class="action-btn toggle-btn" onclick="toggleStatus(${index})">Toggle Paid</button>
          <button class="action-btn delete-btn" onclick="deleteClient(${index})">Delete</button>
        </td>
      `;
      tableBody.appendChild(row);
    });
  }

  document.getElementById('monthlyRevenueDisplay').textContent = `$${totalRevenue.toFixed(2)}`;
  document.getElementById('activeClientsDisplay').textContent = clients.length;
  document.getElementById('appointmentsCountDisplay').textContent = upcomingAppointments;

  checkUpcomingNotifications(clients);
}

// Actions
function toggleStatus(index) {
  playSynthSound('click');
  let clients = getClients();
  
  if (clients[index].status === 'Pending') {
    clients[index].status = 'Paid';
  } else if (clients[index].status === 'Paid') {
    clients[index].status = 'Unpaid';
  } else {
    clients[index].status = 'Paid';
  }

  saveClients(clients);
  renderApp();
}

function deleteClient(index) {
  playSynthSound('click');
  if (confirm('Are you sure you want to remove this client?')) {
    let clients = getClients();
    clients.splice(index, 1);
    saveClients(clients);
    renderApp();
  }
}

clientForm.addEventListener('submit', function (e) {
  e.preventDefault();
  
  const newClient = {
    id: Date.now(),
    name: document.getElementById('clientName').value.trim(),
    phone: document.getElementById('clientPhone').value.trim(),
    status: document.getElementById('clientStatus').value,
    paid: document.getElementById('clientPaid').value || 0,
    appointment: document.getElementById('appointmentDate').value || ''
  };

  const clients = getClients();
  clients.push(newClient);
  saveClients(clients);

  playSynthSound('success');
  closeModal();
  renderApp();
});

document.getElementById('exportBtn').addEventListener('click', function () {
  playSynthSound('click');
  const clients = getClients();
  if (clients.length === 0) {
    alert('No clients available to export.');
    return;
  }

  let csvContent = 'data:text/csv;charset=utf-8,Client Name,Phone Number,Payment Status,Amount Paid,Appointment Time\n';
  clients.forEach(c => {
    const time = c.appointment ? new Date(c.appointment).toLocaleString() : 'N/A';
    csvContent += `"${c.name}","${c.phone || ''}","${c.status}","${c.paid}","${time}"\n`;
  });

  const encodedUri = encodeURI(csvContent);
  const link = document.createElement('a');
  link.setAttribute('href', encodedUri);
  link.setAttribute('download', `${currentUser}_client_index_export.csv`);
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
});

// Guided Tour
const tourSteps = [
  {
    title: "Welcome to ClientIndex! ",
    body: "ClientIndex lets you sign in under your own username to keep your client list completely separate and saved in your browser."
  },
  {
    title: "1. Phone & Details 📞",
    body: "You can now add client phone numbers right alongside their appointment times and payment amounts."
  },
  {
    title: "2. Personal Reminders ⏰",
    body: "Upcoming appointments generate personalized alerts right at the top of your screen."
  }
];

let currentTourStep = 0;
const guideModal = document.getElementById('guideModal');

function showTourStep(index) {
  document.getElementById('tourTitle').textContent = tourSteps[index].title;
  document.getElementById('tourBody').textContent = tourSteps[index].body;
  if (index === tourSteps.length - 1) {
    document.getElementById('nextTourBtn').textContent = "Finish Tour 🎉";
  } else {
    document.getElementById('nextTourBtn').textContent = "Next Step ➡️";
  }
}

document.getElementById('guideBtn').addEventListener('click', () => {
  playSynthSound('click');
  currentTourStep = 0;
  showTourStep(currentTourStep);
  guideModal.style.display = 'flex';
});

document.getElementById('closeGuideBtn').addEventListener('click', () => {
  playSynthSound('click');
  guideModal.style.display = 'none';
});

document.getElementById('nextTourBtn').addEventListener('click', () => {
  playSynthSound('click');
  currentTourStep++;
  if (currentTourStep < tourSteps.length) {
    showTourStep(currentTourStep);
  } else {
    guideModal.style.display = 'none';
  }
});

document.addEventListener('DOMContentLoaded', checkUserAuth);
