# my personal website

### How to build the boilerplate from scratch (MacOS):

- install https://github.com/nvm-sh/nvm if not already installed
  - `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash` or newer version from [the website](https://github.com/nvm-sh/nvm).
  - install Node.js 20
    - `nvm install node`
    - `nvm ls`
    - `nvm use 20`
    - `node -v`
- `npm create vite@latest` # choose react, javascript, name frontend
- `cd frontend`
- `rm -rf node_modules`
- `npm install`
- `npm install bootstrap`
- `npm install react-router-dom`
- `npm run dev`
  - http://localhost:5173/
- `npm run build`

  - `cd dist`
  - if MacOS Ventura: `brew install php`
  - `php -S localhost:2222` or `python -m http.server 2222`
  - http://localhost:2222/

  The actual code and the router is roughly based off the fronted parts of my other projects like [https://github.com/EL-CL/voiceprint-demo](https://github.com/EL-CL/voiceprint-demo).

### Bootstrap CSS formatting:

- https://getbootstrap.com/docs/5.0/
