
const CACHE_NAME = 'math-app-v2';
const urlsToCache = [
  './',
  './index.html',
  './manifest.json',
  './icon-192.png',
  './icon-512.png',
  './og-image.png'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(urlsToCache)).catch(()=>{})
  );
  self.skipWaiting();
});

self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(keys => Promise.all(keys.map(k => { if(k!==CACHE_NAME) return caches.delete(k) })))
  );
  self.clients.claim();
});

self.addEventListener('fetch', event => {
  if(event.request.url.includes('supabase') || event.request.url.includes('cdn.jsdelivr.net') || event.request.url.includes('vercel')){
    return;
  }
  event.respondWith(
    fetch(event.request).catch(()=> caches.match(event.request).then(r=> r || caches.match('./index.html')))
  );
});
