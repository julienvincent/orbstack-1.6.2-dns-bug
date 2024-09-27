## OrbStack 1.6.2 Reproduce Project

This is a simple repo demonstrating how to reproduce a dns bug first presenting in OrbStack 1.6.2 and still present in
at least OrbStack 1.7.4.

### Steps to reproduce

Make sure you are on an OrbStack version >=1.6.2

1) Run the docker compose stack

```bash
docker compose up -d
```

2) Attempt to make an http call to `webserver.test`

```bash
docker run --rm -it \
  --network demo \
  --dns 172.6.0.100 \
  node:22.9.0-alpine3.20 \
    node -e 'fetch("http://webserver.test").then(res => res.text()).then(console.log)'
```

This should print out something like the below, reproducing the issue.

```
node:internal/deps/undici/undici:13185
      Error.captureStackTrace(err);
            ^

TypeError: fetch failed
    at node:internal/deps/undici/undici:13185:13
    at process.processTicksAndRejections (node:internal/process/task_queues:105:5) {
  [cause]: Error: getaddrinfo ENOTFOUND webserver.test
      at GetAddrInfoReqWrap.onlookupall [as oncomplete] (node:dns:120:26) {
    errno: -3008,
    code: 'ENOTFOUND',
    syscall: 'getaddrinfo',
    hostname: 'webserver.test'
  }
}

Node.js v22.9.0
```

3) Now run the same but without the `-alpine3.20` tag

```bash
docker run --rm -it \
  --network demo \
  --dns 172.6.0.100 \
  node:22.9.0 \
    node -e 'fetch("http://webserver.test").then(res => res.text()).then(console.log)'
```

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

---

Performing all of the above steps on 1.6.1 should produce the same successful nginx output in step 2. as is shown in
step 3.
