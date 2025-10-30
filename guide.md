# Integrating Lingo.dev CLI with Electron

This guide will walk you through integrating the [Lingo.dev CLI](https://lingo.dev/en/cli) with a new [Electron](https://www.electronjs.org/) project. We will use the popular [Electron Forge](https://www.electronforge.io/) toolkit with its React template and the [React-Intl](https://formatjs.io/docs/react-intl/) library to manage and display our translations.

By the end of this tutorial, you'll have a running Electron desktop application that loads translations from Lingo.dev.

## Prerequisites

Before you begin, you need to have the following installed:
* [Node.js](https://nodejs.org/) (which includes npm)
* A free [Lingo.dev account](https://app.lingo.dev/signup)

---

## Step 1: Create a new Electron project

We'll use `create-electron-app` with the `webpack-react` template to bootstrap our project. This sets up Electron, React, and all the necessary build tooling.

Open your terminal and run the following commands:

```bash
# Create a new Electron + React project
npx create-electron-app@latest my-electron-app --template=webpack-react

# Navigate into the new project directory
cd my-electron-app

This will create a new folder my-electron-app with a basic React application ready to run inside Electron.
Step 2: Install and configure Lingo.dev CLI
Next, let's add the Lingo.dev CLI to the project.
# Install the CLI as a dev dependency
npm install @lingo-dev/cli --save-dev

Now, initialize the CLI. This command will create a lingo.json configuration file at the root of your project.
# Initialize Lingo.dev
npx @lingo-dev/cli init

You will be prompted to answer a few questions. For this guide, use the following answers:
 * What format do you want to use?
   * Select react-intl (nested JSON objects).
 * Where do you want to store your translation files?
   * Enter: src/locales/{locale}.json
 * What is your source locale?
   * Enter: en
 * Do you want to upload existing translation files?
   * Select: No
This will create a lingo.json file that looks like this:
lingo.json
{
  "format": "react-intl",
  "path": "src/locales/{locale}.json",
  "sourceLocale": "en"
}

This configuration tells the Lingo.dev CLI to:
 * Use the react-intl format (e.g., { "home": { "title": "Hello" } }).
 * Store translation files inside the src/locales directory, named by their locale code (e.g., en.json, es.json).
 * Treat en (English) as the source language.
Step 3: Add content in Lingo.dev
Now, let's create the content we want to translate. Go to your Lingo.dev dashboard and create a new Project.
 * Add your source language (English).
 * Add a target language. We'll use Spanish (es) for this example.
 * Go to the Content tab and add the following two content keys:
   * Key: home.title
     * English: Hello World!
     * Spanish: ¡Hola Mundo!
   * Key: home.description
     * English: Welcome to your multilingual Electron app.
     * Spanish: Bienvenido a tu aplicación de Electron multilingüe.
Your Lingo.dev project is now ready.
Step 4: Install React-Intl
To use our translations in React, we need to install the react-intl package.
npm install react-intl

Step 5: Pull translations
Let's pull the content we just created from Lingo.dev. Run the pull command:
npx @lingo-dev/cli pull

You will be prompted to link your local project to the Lingo.dev project you created in Step 3. After linking, the CLI will pull the content and create two new files:
 * src/locales/en.json
 * src/locales/es.json
If you inspect src/locales/en.json, you'll see the nested JSON structure we defined:
src/locales/en.json
{
  "home": {
    "title": "Hello World!",
    "description": "Welcome to your multilingual Electron app."
  }
}

Step 6: Configure React-Intl Provider
Now we need to make react-intl available to our React application. We do this by modifying the main renderer entry point to wrap our <App /> component with an <IntlProvider>.
Open src/renderer.js and replace its contents with the following code. This code imports our new translation files, selects a locale, and configures the provider.
src/renderer.js
import React from 'react';
import { createRoot } from 'react-dom/client';
import { IntlProvider } from 'react-intl';
import App from './app';

// Import translation files
import messages_en from './locales/en.json';
import messages_es from './locales/es.json';

// Setup messages object
const messages = {
    'en': messages_en,
    'es': messages_es
};

// Set the locale
// -------------------------------------------------
const locale = 'en'; // <-- Change this to 'es' to test!
// -------------------------------------------------

// Find the root element
const container = document.getElementById('root');
const root = createRoot(container);

// Render the app with the IntlProvider
root.render(
    <IntlProvider locale={locale} messages={messages[locale]}>
        <App />
    </IntlProvider>
);

> Note: In a real-world application, you would dynamically determine the locale using navigator.language or Electron's app.getLocale() method. For this guide, we'll hardcode it to keep things simple.
> 
Step 7: Render translated content
With the provider configured, we can now use react-intl's components to render our content.
Open src/app.jsx and replace its contents with the following:
src/app.jsx
import React from 'react';
import { FormattedMessage } from 'react-intl';

const App = () => (
  <>
    <h1>
      <FormattedMessage
        id="home.title"
        defaultMessage="Hello World!"
      />
    </h1>
    <p>
      <FormattedMessage
        id="home.description"
        defaultMessage="Welcome to your multilingual Electron app."
      />
    </p>
  </>
);

export default App;

We've replaced the static text with the <FormattedMessage /> component.
 * The id prop matches the key we created in Lingo.dev (home.title).
 * The defaultMessage is a good practice fallback, ensuring your app still displays text if a translation is missing.
Step 8: Run the app
That's it! Let's run the app in development mode.
npm start

After a moment, your Electron application window will appear, displaying the English content pulled from Lingo.dev.
Step 9: Test other languages
Let's test our Spanish translation.
 * Go back to src/renderer.js.
 * Change the locale variable from 'en' to 'es':
   // Set the locale
// -------------------------------------------------
const locale = 'es'; // <-- Change this to 'es' to test!
// -------------------------------------------------

 * Save the file. The Electron Forge development server will automatically hot-reload your application.
Your app window will update to show the Spanish content.
Next steps
You now have a fully working, multilingual Electron and React application connected to Lingo.dev.
From here, you can:
 * Add more languages in the Lingo.dev dashboard.
 * Add more content keys.
 * Run npx @lingo-dev/cli pull to sync your app with the new content.
 * Read the Lingo.dev CLI documentation to learn about other commands like push and sync.
<!-- end list -->

