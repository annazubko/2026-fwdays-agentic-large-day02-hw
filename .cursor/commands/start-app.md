# Command: start-app

## Purpose

Start the Excalidraw development server.

## When to use

When you need to run the application locally for development or manual testing.

## Prompt

Run the Excalidraw development server by executing:

```bash
yarn start
```

This starts the dev server at **http://localhost:3001**.

If `$ARGUMENTS` is `production`, run instead:

```bash
yarn start:production
```

This builds and serves at **http://localhost:5001**.

Report the URL and confirm the server started successfully.