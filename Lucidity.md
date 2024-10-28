# Lucidity Node.js Installation Guide

This guide will help you install and set up Lucidity for your Node.js project.

## Prerequisites

- Node.js v20.12.2 or higher
- A Node.js project set up to use ESM (ECMAScript modules)

## Installation

1. Create a new Node.js project or navigate to an existing one.

2. Install Lucidity Core:

   ```bash
   npm install lucidity-core
   ```

## Configuration

1. Create a `lucid.config.js` or `lucid.config.ts` file in the root of your project.

2. Add the minimum required configuration:

   ```javascript
   export default {
     collections: {},
     locales: ['en'],
   }
   ```

## Importing and Starting Lucidity

1. In your main application file (e.g. `index.js`), import and start Lucidity:

   ```javascript
   import { lucid } from 'lucidity-core';

   async function main() {
     await lucid.start();
     console.log('Lucidity started successfully');
   }

   main().catch(console.error);
   ```

2. Run your application:

   ```bash
   node index.js
   ```

## Accessing the Admin Interface

Once Lucidity is running:

1. Open a web browser and navigate to `http://localhost:8080/login`
2. Use the default credentials:
   - Username: `admin`
   - Password: `password`

**Note**: You will be prompted to change the default password on first login.

## Additional Configuration

- **Collections**: Define your content collections in the `lucid.config.js` file.
- **Locales**: Add support for multiple languages by expanding the `locales` array.
- **Email and Media Strategies**: Configure these in the `lucid.config.js` file as needed.

## Best Practices

- Always use ESM syntax in your project when working with Lucidity.
- Keep your `lucid.config.js` file up-to-date as your project evolves.
- Regularly check for updates to Lucidity Core and other dependencies.

For more advanced features and configurations, please consult the official Lucidity documentation.
```

