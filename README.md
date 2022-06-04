# Base for Vite HMR development on Docker

This is a base project for using Vite on a Docker container for development
using the HMR on a NGINX reverse proxy.

There is two containers:
- reverse-proxy (NGINX server on port 80)
- ui (vite app)

The 3000 port is used internally for the UI communication with the NGINX proxy.
The 80 port is used externally for the main app and port 3001 for the HMR.


More details on the [Step by Step](STEPS.md) file.

Feel free to play like you want with this!
I learned a lot making it, I hope this can be useful for someone too!
