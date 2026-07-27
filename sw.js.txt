
const CACHE_NAME = 'math-app-v1';
const urlsToCache = [
  '/',
  '/manifest.json'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(urlsToCache))
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
  // Bypass supabase and external CDNs from cache
  if(event.request.url.includes('supabase') || event.request.url.includes('cdn.jsdelivr.net')){
    return;
  }
  event.respondWith(
    caches.match(event.request).then(resp => {
      return resp || fetch(event.request).then(r => {
        // Cache html
        if(event.request.method === 'GET' && event.request.url.startsWith(self.location.origin)){
          const clone = r.clone();
          caches.open(CACHE_NAME).then(c => c.put(event.request, clone));
        }
        return r;
      }).catch(()=> caches.match('/'))
    })
  );
});
