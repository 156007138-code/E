<!DOCTYPE html>
<!--
Created using JS Bin
http://jsbin.com

Copyright (c) 2025 by anonymous (http://jsbin.com/mogofayiku/1/edit)

Released under the MIT license: http://jsbin.mit-license.org
-->
<meta name="robots" content="noindex">
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RIVS BROWSER</title>
    <!-- SECURITY ENHANCEMENT: Prevents sending this page's URL as a referrer to external sites. -->
    <meta name="referrer" content="no-referrer">
    
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Define default CSS variables for dynamic theming */
        :root {
            --color-primary: #4f46e5;      /* Default primary (Indigo-600) */
            --color-background: #1f2937;   /* Default dark background (Gray-800) */
        }

        /* Custom styles for the scrollbar and full-height layout */
        body {
            font-family: 'Inter', sans-serif;
            /* Use CSS variable for background color */
            background-color: var(--color-background); 
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            transition: background-color 0.3s ease, padding 0.3s ease;
        }
        #content-frame {
            flex-grow: 1; 
            width: 100%;
            border: none;
            background-color: #ffffff; /* Light background for the content area */
            border-radius: 0.5rem;
        }
        /* Style the placeholder container */
        #placeholder-container {
            flex-grow: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #9ca3af;
            font-size: 1.5rem;
            text-align: center;
            padding: 2rem;
        }

        /* FULLSCREEN MODE STYLES */
        body.is-fullscreen {
            padding: 0 !important;
            margin: 0 !important;
            min-height: 100vh;
            height: 100vh;
            overflow: hidden; 
        }

        /* Hide header and controls when in fullscreen mode */
        body.is-fullscreen #header-area,
        body.is-fullscreen #controls-area {
            display: none !important;
        }

        /* Expand viewer container to full screen */
        body.is-fullscreen #viewer-container {
            max-width: 100vw !important;
            width: 100vw !important;
            height: 100vh !important;
            margin: 0 !important;
            padding: 0 !important;
            border-radius: 0 !important;
            transition: all 0.3s ease;
        }

        /* Ensure iframe fills its new container */
        body.is-fullscreen #content-frame {
            height: 100vh !important;
            border-radius: 0 !important;
        }

        /* Modal specific styles */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
    </style>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        // Map Tailwind colors to CSS variables for dynamic theming
                        'primary': 'var(--color-primary)', 
                        'secondary': '#1e293b', // Static secondary color
                    },
                }
            }
        }
    </script>
</head>
<body class="p-4 sm:p-6">

    <div class="max-w-7xl w-full mx-auto" id="header-area">
        <h1 class="text-3xl font-extrabold text-white mb-2">RIVS BROWSER</h1>
        <p class="text-gray-400 mb-6">Enter a search query or a full URL to view it below.</p>
    </div>

    <div class="max-w-7xl w-full mx-auto flex space-x-3 mb-6" id="controls-area">
        <input 
            type="text" 
            id="url-input" 
            placeholder="Search or enter a URL (e.g., example.com)" 
            class="flex-1 p-3 border-2 border-gray-700 bg-gray-800 text-white rounded-lg focus:ring-primary focus:border-primary transition duration-150 shadow-lg"
            onkeydown="if(event.key === 'Enter') navigateOrSearch()"
        >
        <button 
            id="go-button" 
            onclick="navigateOrSearch()" 
            class="px-6 py-3 bg-primary text-white font-semibold rounded-lg shadow-xl hover:bg-indigo-700 transition duration-150 transform hover:scale-[1.02] active:scale-[0.98] focus:outline-none focus:ring-4 focus:ring-primary focus:ring-opacity-50"
        >
            Go
        </button>
        
        <!-- Clear Session Button -->
        <button 
            id="clear-button" 
            onclick="clearSession()" 
            class="p-3 bg-red-600 text-white rounded-lg shadow-xl hover:bg-red-700 transition duration-150 transform hover:scale-[1.02] active:scale-[0.98] focus:outline-none focus:ring-4 focus:ring-red-500 focus:ring-opacity-50"
            title="Clear Session"
        >
            <!-- X Mark Icon -->
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="w-6 h-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
            </svg>
        </button>
        
        <!-- Settings Button -->
        <button 
            id="settings-button" 
            onclick="toggleSettingsModal(true)" 
            class="p-3 bg-gray-600 text-white rounded-lg shadow-xl hover:bg-gray-700 transition duration-150 transform hover:scale-[1.02] active:scale-[0.98] focus:outline-none focus:ring-4 focus:ring-gray-500 focus:ring-opacity-50"
            title="Customize Settings"
        >
             <!-- Gear Icon -->
             <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="w-6 h-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.294.608 3.29 0z" />
                <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 12a3.75 3.75 0 11-7.5 0 3.75 3.75 0 017.5 0z" />
            </svg>
        </button>
        
        <!-- Fullscreen Button -->
        <button 
            id="fullscreen-button" 
            onclick="toggleFullscreen()" 
            class="p-3 bg-gray-600 text-white rounded-lg shadow-xl hover:bg-gray-700 transition duration-150 transform hover:scale-[1.02] active:scale-[0.98] focus:outline-none focus:ring-4 focus:ring-gray-500 focus:ring-opacity-50"
            title="Toggle Fullscreen"
        >
            <!-- Maximize Icon (Initial state) -->
            <svg id="fullscreen-icon" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="w-6 h-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 3.75v4.5m0-4.5h4.5m-4.5 0L9 9M20.25 3.75h-4.5m4.5 0v4.5m0-4.5L15 9M5.25 16.5v4.5m0-4.5h4.5m-4.5 0L9 15M20.25 16.5h-4.5m4.5 0v4.5m0-4.5L15 15" />
            </svg>
        </button>
    </div>

    <!-- The container for the content -->
    <div class="max-w-7xl w-full mx-auto flex-grow bg-gray-900 rounded-xl shadow-2xl p-0 overflow-hidden relative" id="viewer-container">
        <!-- Iframe to load the content -->
        <iframe 
            id="content-frame" 
            title="Contained Web Content" 
            class="hidden"
            <!-- The restrictive 'sandbox' attribute has been removed to allow more websites to load by helping to bypass common iframe restrictions (like X-Frame-Options). -->
        ></iframe>

        <!-- Placeholder/Welcome Screen -->
        <div id="placeholder-container">
            <div class="p-8 bg-gray-800 rounded-xl shadow-inner max-w-sm">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10 mx-auto text-primary mb-3" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM4.332 9.4a1 1 0 01.192-.58L6.8 6.643a1 1 0 01.708-.277h4.945c.31 0 .603.117.828.318l2.062 1.843a1 1 0 01.192.581V12.5a1 1 0 01-1 1H5.332a1 1 0 01-1-1V9.4z" clip-rule="evenodd" />
                </svg>
                <p class="text-gray-400 text-lg font-medium">Welcome to RIVS BROWSER!</p>
                <p class="text-sm text-gray-500 mt-1">Search or enter a URL above to start viewing content privately within this window.</p>
            </div>
        </div>
    </div>

    <!-- Settings Modal (Initially Hidden) -->
    <div id="settings-modal" class="modal-overlay hidden">
        <div class="bg-gray-800 p-6 rounded-xl shadow-2xl w-full max-w-sm">
            <h2 class="text-2xl font-bold text-white mb-4 border-b border-gray-700 pb-2">Appearance & Function Settings</h2>
            
            <div class="space-y-4">
                
                <!-- Primary Color Picker -->
                <div>
                    <label for="primary-color-input" class="block text-sm font-medium text-gray-300 mb-1">Theme Color (Buttons/Accents)</label>
                    <div class="flex items-center space-x-3">
                        <input 
                            type="color" 
                            id="primary-color-input" 
                            class="w-10 h-10 border-2 border-gray-600 rounded-lg cursor-pointer"
                            onchange="updateColor('primary', this.value)"
                        >
                        <span id="primary-color-hex" class="text-gray-400 text-sm">#4f46e5</span>
                    </div>
                </div>

                <!-- Background Color Picker -->
                <div>
                    <label for="background-color-input" class="block text-sm font-medium text-gray-300 mb-1">Background Color</label>
                    <div class="flex items-center space-x-3">
                        <input 
                            type="color" 
                            id="background-color-input" 
                            class="w-10 h-10 border-2 border-gray-600 rounded-lg cursor-pointer"
                            onchange="updateColor('background', this.value)"
                        >
                        <span id="background-color-hex" class="text-gray-400 text-sm">#1f2937</span>
                    </div>
                </div>

                <!-- Default Search Engine Selector -->
                <div>
                    <label for="search-engine-select" class="block text-sm font-medium text-gray-300 mb-1">Default Search Engine</label>
                    <select 
                        id="search-engine-select" 
                        class="w-full p-2 border-2 border-gray-600 bg-gray-700 text-white rounded-lg focus:ring-primary focus:border-primary transition duration-150"
                        onchange="updateSearchEngine(this.value)"
                    >
                        <option value="google">Google (iframe friendly)</option>
                        <option value="duckduckgo">DuckDuckGo (More Private)</option>
                        <option value="bing">Bing</option>
                        <option value="startpage">StartPage (Secure Search)</option>
                        <option value="yahoo">Yahoo</option>
                    </select>
                </div>
                
                <!-- Custom Home Page URL Input -->
                <div>
                    <label for="home-page-input" class="block text-sm font-medium text-gray-300 mb-1">Custom Home Page URL (e.g., wikipedia.org)</label>
                    <input 
                        type="text" 
                        id="home-page-input" 
                        placeholder="Leave blank for a blank page" 
                        class="w-full p-2 border-2 border-gray-600 bg-gray-700 text-white rounded-lg focus:ring-primary focus:border-primary transition duration-150"
                        oninput="updateHomePage(this.value)"
                    >
                    <p class="text-xs text-gray-500 mt-1">Press 'Go' on an empty search bar to load this page.</p>
                </div>

            </div>

            <button 
                onclick="toggleSettingsModal(false)" 
                class="mt-6 w-full py-2 bg-primary text-white font-semibold rounded-lg shadow-xl hover:bg-indigo-700 transition duration-150"
            >
                Close
            </button>
        </div>
    </div>


    <script>
        // SVG paths for the Fullscreen toggle button
        const EXPAND_ICON = `<path stroke-linecap="round" stroke-linejoin="round" d="M3.75 3.75v4.5m0-4.5h4.5m-4.5 0L9 9M20.25 3.75h-4.5m4.5 0v4.5m0-4.5L15 9M5.25 16.5v4.5m0-4.5h4.5m-4.5 0L9 15M20.25 16.5h-4.5m4.5 0v4.5m0-4.5L15 15" />`;
        const COLLAPSE_ICON = `<path stroke-linecap="round" stroke-linejoin="round" d="M15 15l-4.5 4.5M9 9l4.5 4.5M20.25 9l-4.5 4.5m-8.25 1.5h8.25m-4.5 4.5h.008v.008h-.008v-.008z" />`;

        const STORAGE_KEY = 'rivs_browser_settings';
        const DEFAULT_COLORS = {
            primary: '#4f46e5',
            background: '#1f2937'
        };
        const DEFAULT_SEARCH_ENGINE = 'google';
        const DEFAULT_HOME_PAGE = 'about:blank'; // Use blank page as default home

        // Map of available search engines and their URL formats
        const SEARCH_ENGINES = {
            google: { url: 'https://www.google.com/search?igu=1&q=', name: 'Google' },
            duckduckgo: { url: 'https://duckduckgo.com/?q=', name: 'DuckDuckGo (More Private)' },
            bing: { url: 'https://www.bing.com/search?q=', name: 'Bing' },
            startpage: { url: 'https://www.startpage.com/sp/search?query=', name: 'StartPage (Secure Search)' },
            yahoo: { url: 'https://search.yahoo.com/search?p=', name: 'Yahoo' }
        };


        /**
         * Retrieves the current settings object from local storage, merging with defaults.
         * @returns {object} The current settings.
         */
        function getCurrentSettings() {
             let settings = { 
                ...DEFAULT_COLORS,
                searchEngine: DEFAULT_SEARCH_ENGINE,
                homePage: DEFAULT_HOME_PAGE // Include new default
            };
            try {
                const storedSettings = localStorage.getItem(STORAGE_KEY);
                if (storedSettings) {
                    // Merge stored settings with defaults to ensure all keys exist
                    const loaded = JSON.parse(storedSettings);
                    settings = { ...settings, ...loaded };
                }
            } catch (e) {
                console.error("Could not load settings from localStorage:", e);
                // Fallback to defaults
            }
            return settings;
        }

        /**
         * Loads and applies colors and settings from localStorage, or uses defaults.
         */
        function initializeSettings() {
            const settings = getCurrentSettings();
            
            // Apply colors to the UI and CSS variables
            applyColors(settings);

            // Set color picker inputs to match
            document.getElementById('primary-color-input').value = settings.primary;
            document.getElementById('primary-color-hex').textContent = settings.primary;

            document.getElementById('background-color-input').value = settings.background;
            document.getElementById('background-color-hex').textContent = settings.background;
            
            // Set search engine dropdown
            const searchSelect = document.getElementById('search-engine-select');
            if (searchSelect) {
                searchSelect.value = settings.searchEngine;
            }
            
            // Set home page input (New)
            const homePageInput = document.getElementById('home-page-input');
            if (homePageInput) {
                // Display the current home page URL in the input field (leave blank if it's the default blank page)
                homePageInput.value = (settings.homePage === DEFAULT_HOME_PAGE) ? "" : settings.homePage; 
            }
        }

        /**
         * Applies the given color settings to CSS variables.
         * @param {object} colors - Object containing primary and background color hex codes.
         */
        function applyColors(colors) {
            const root = document.documentElement;
            root.style.setProperty('--color-primary', colors.primary);
            root.style.setProperty('--color-background', colors.background);
        }

        /**
         * Updates a specific color, saves it, and applies it.
         * @param {string} key - 'primary' or 'background'.
         * @param {string} value - New hex color code.
         */
        function updateColor(key, value) {
            try {
                // Load current settings object
                const storedSettings = getCurrentSettings();
                storedSettings[key] = value;
                
                // Save and apply
                localStorage.setItem(STORAGE_KEY, JSON.stringify(storedSettings));
                applyColors(storedSettings);

                // Update hex display text
                document.getElementById(`${key}-color-hex`).textContent = value;

            } catch (e) {
                console.error("Could not save or apply color:", e);
            }
        }

        /**
         * Updates the default search engine choice.
         * @param {string} engineKey - The key of the selected search engine (e.g., 'google').
         */
        function updateSearchEngine(engineKey) {
             try {
                // Load current settings object
                const storedSettings = getCurrentSettings();
                storedSettings.searchEngine = engineKey;
                
                // Save
                localStorage.setItem(STORAGE_KEY, JSON.stringify(storedSettings));

            } catch (e) {
                console.error("Could not save search engine setting:", e);
            }
        }
        
        /**
         * Updates the custom home page URL.
         * @param {string} url - The new URL string.
         */
        function updateHomePage(url) {
            try {
                const storedSettings = getCurrentSettings();
                
                let normalizedUrl = url.trim();
                
                // If input is empty, reset to about:blank default
                if (!normalizedUrl) {
                    normalizedUrl = DEFAULT_HOME_PAGE;
                }
                
                // Prepend 'https://' if no protocol is present for non-blank URLs
                if (normalizedUrl !== DEFAULT_HOME_PAGE && !normalizedUrl.match(/^https?:\/\//i)) {
                    normalizedUrl = 'https://' + normalizedUrl;
                }

                storedSettings.homePage = normalizedUrl;
                localStorage.setItem(STORAGE_KEY, JSON.stringify(storedSettings));
                
                // If they are on the placeholder screen, update the placeholder's hint
                updatePlaceholderHint(normalizedUrl);

            } catch (e) {
                console.error("Could not save home page setting:", e);
            }
        }
        
        /**
         * Updates the placeholder text based on the current home page setting.
         */
        function updatePlaceholderHint(url) {
            const placeholderText = document.querySelector('#placeholder-container .text-sm');
            // Check current home page settings
            const homeUrl = url || getCurrentSettings().homePage;
            
            if (placeholderText) {
                if (homeUrl && homeUrl !== DEFAULT_HOME_PAGE) {
                    // Only show the hint if a custom home page is set
                    placeholderText.textContent = `Search, enter a URL, or press 'Go' on an empty box to visit your home page: ${homeUrl}`;
                } else {
                     placeholderText.textContent = `Search or enter a URL above to start viewing content privately within this window.`;
                }
            }
        }


        /**
         * Toggles the visibility of the settings modal.
         * @param {boolean} show - True to show, False to hide.
         */
        function toggleSettingsModal(show) {
            const modal = document.getElementById('settings-modal');
            if (show) {
                modal.classList.remove('hidden');
            } else {
                modal.classList.add('hidden');
            }
        }

        /**
         * Clears the current browsing session: resets the iframe, input, and shows the placeholder.
         */
        function clearSession() {
            const inputElement = document.getElementById('url-input');
            const iframeElement = document.getElementById('content-frame');
            const placeholderElement = document.getElementById('placeholder-container');

            // Reset the iframe source and hide it
            iframeElement.src = 'about:blank';
            iframeElement.classList.add('hidden');

            // Clear the input field
            inputElement.value = '';

            // Show the placeholder/welcome screen
            placeholderElement.classList.remove('hidden');
            updatePlaceholderHint(); // Reset hint text if needed
        }


        /**
         * Core logic to determine if the input is a URL or a search query,
         * construct the target URL, and load it into the iframe.
         */
        function navigateOrSearch() {
            const inputElement = document.getElementById('url-input');
            const iframeElement = document.getElementById('content-frame');
            const placeholderElement = document.getElementById('placeholder-container');
            let query = inputElement.value.trim();
            const currentSettings = getCurrentSettings();

            let targetUrl = '';
            
            if (!query) {
                // If input is empty, load the configured Home Page
                targetUrl = currentSettings.homePage; 
                if (!targetUrl || targetUrl === DEFAULT_HOME_PAGE) {
                     // If Home Page is also blank, just return.
                    return; 
                }
            } else {
                // 1. Check if the input looks like a simple URL
                const urlRegex = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/i;

                if (urlRegex.test(query)) {
                    // It looks like a domain or URL
                    targetUrl = query;
                    // Prepend 'https://' if no protocol is present
                    if (!targetUrl.match(/^https?:\/\//i)) {
                        targetUrl = 'https://' + targetUrl;
                    }
                } else {
                    // 2. Treat it as a search query using the configured engine
                    const encodedQuery = encodeURIComponent(query);
                    
                    const engineKey = currentSettings.searchEngine || DEFAULT_SEARCH_ENGINE;
                    const engine = SEARCH_ENGINES[engineKey] || SEARCH_ENGINES[DEFAULT_SEARCH_ENGINE];

                    targetUrl = `${engine.url}${encodedQuery}`; 
                }
            }


            // Load the URL into the iframe
            iframeElement.src = targetUrl;

            // Show the iframe and hide the placeholder
            iframeElement.classList.remove('hidden');
            placeholderElement.classList.add('hidden');

            // Optional: Clear input after navigating
            // inputElement.value = '';
        }

        /**
         * Toggles the browser into and out of fullscreen mode by applying a CSS class
         * to the body and updating the button icon.
         */
        function toggleFullscreen() {
            const body = document.body;
            const iconContainer = document.getElementById('fullscreen-icon');
            
            body.classList.toggle('is-fullscreen');

            // Update button icon based on the new state
            if (body.classList.contains('is-fullscreen')) {
                // Currently fullscreen, show the collapse icon
                iconContainer.innerHTML = COLLAPSE_ICON;
            } else {
                // Not fullscreen, show the expand icon
                iconContainer.innerHTML = EXPAND_ICON;
            }
        }

        // Run on page load
        document.addEventListener('DOMContentLoaded', () => {
            initializeSettings(); // Load and apply custom colors and settings
            
            const input = document.getElementById('url-input');
            input.addEventListener('keydown', (event) => {
                if (event.key === 'Enter') {
                    navigateOrSearch();
                }
            });
            // Update the initial placeholder hint based on home page setting
            updatePlaceholderHint(); 
        });

    </script>
</body>
</html>
