const http = require('http');
const { WebSocketServer } = require('ws');
const path = require('path');
const fs = require('fs');

const PORT = process.env.PORT || 3000;

// ── Serveur HTTP pour les fichiers statiques ──
const server = http.createServer((req, res) => {
  let filePath = path.join(__dirname, 'public');

  if (req.url === '/' || req.url === '/index.html') {
    filePath = path.join(filePath, 'index.html');
  } else if (req.url === '/viewer' || req.url === '/viewer.html') {
    filePath = path.join(filePath, 'viewer.html');
  } else {
    res.writeHead(404);
    return res.end('Not found');
  }

  const ext = path.extname(filePath);
  const mime = { '.html': 'text/html', '.js': 'application/javascript', '.css': 'text/css' };
  res.writeHead(200, { 'Content-Type': mime[ext] || 'text/plain' });
  fs.createReadStream(filePath).pipe(res);
});

// ── WebSocket ──
const wss = new WebSocketServer({ server });

// Map des caméras actives : camId -> { ws, number }
const cameras = new Map();
// Set des viewers
const viewers = new Set();
let camCounter = 0;

function broadcastCameraList() {
  const list = [];
  cameras.forEach((data, camId) => {
    list.push({ id: camId, number: data.number });
  });
  const msg = JSON.stringify({ type: 'camera-list', cameras: list });
  viewers.forEach(ws => { if (ws.readyState === 1) ws.send(msg); });
}

wss.on('connection', (ws) => {
  let role = null;
  let myCamId = null;

  ws.on('message', (raw) => {
    let msg;
    try { msg = JSON.parse(raw); } catch { return; }

    switch (msg.type) {

      // ── Caméra s'enregistre ──
      case 'register-camera': {
        role = 'camera';
        camCounter++;
        myCamId = 'cam-' + Date.now();
        cameras.set(myCamId, { ws, number: camCounter });
        ws.send(JSON.stringify({ type: 'registered', camId: myCamId, number: camCounter }));
        broadcastCameraList();
        break;
      }

      // ── Viewer demande la liste ──
      case 'register-viewer': {
        role = 'viewer';
        viewers.add(ws);
        broadcastCameraList();
        break;
      }

      // ── Viewer veut se connecter à une cam ──
      case 'request-connect': {
        // msg.camId, msg.viewerOffer (SDP)
        const cam = cameras.get(msg.camId);
        if (!cam || cam.ws.readyState !== 1) {
          ws.send(JSON.stringify({ type: 'error', message: 'Caméra non disponible' }));
          return;
        }
        // Transmettre l'offre à la caméra avec référence au viewer
        cam.ws.send(JSON.stringify({
          type: 'viewer-offer',
          viewerWsId: msg.viewerWsId,
          offer: msg.offer
        }));
        // Stocker référence viewer → ws pour la réponse
        ws._wsId = msg.viewerWsId;
        break;
      }

      // ── Caméra répond au viewer ──
      case 'camera-answer': {
        // Trouver le viewer par wsId
        viewers.forEach(vws => {
          if (vws._wsId === msg.viewerWsId && vws.readyState === 1) {
            vws.send(JSON.stringify({ type: 'camera-answer', answer: msg.answer }));
          }
        });
        break;
      }

      // ── ICE candidates ──
      case 'ice-candidate': {
        if (msg.to === 'camera') {
          const cam = cameras.get(msg.camId);
          if (cam && cam.ws.readyState === 1) {
            cam.ws.send(JSON.stringify({ type: 'ice-candidate', candidate: msg.candidate, viewerWsId: msg.viewerWsId }));
          }
        } else if (msg.to === 'viewer') {
          viewers.forEach(vws => {
            if (vws._wsId === msg.viewerWsId && vws.readyState === 1) {
              vws.send(JSON.stringify({ type: 'ice-candidate', candidate: msg.candidate }));
            }
          });
        }
        break;
      }

      // ── Commande flip ──
      case 'flip-camera': {
        const cam = cameras.get(msg.camId);
        if (cam && cam.ws.readyState === 1) {
          cam.ws.send(JSON.stringify({ type: 'flip' }));
        }
        break;
      }
    }
  });

  ws.on('close', () => {
    if (role === 'camera' && myCamId) {
      cameras.delete(myCamId);
      broadcastCameraList();
    }
    if (role === 'viewer') {
      viewers.delete(ws);
    }
  });
});

server.listen(PORT, () => console.log(`HomeCam server running on port ${PORT}`));
