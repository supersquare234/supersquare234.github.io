<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Web Dashers</title>
    <link rel="icon" type="image/x-icon" href="assets/favicon.ico">
    <link rel="stylesheet" href="assets/style.css">

    <meta name="description" content="Web Dashers is a fan-made browser mod of the Geometry Dash Web Demo. It features the main levels, a built-in level creator, and a level search to play online levels. The project is actively in development, with more features and updates planned.">
    <meta name="theme-color" content="#C1FB3D">

    <meta property="og:type" content="website" />
    <meta property="og:url" content="https://web-dashers.github.io/" />
    <meta property="og:title" content="Web Dashers" />
    <meta property="og:description" content="Web Dashers is a fan-made browser mod of the Geometry Dash Web Demo. It features the main levels, a built-in level creator, and a level search to play online levels. The project is actively in development, with more features and updates planned." />
    <meta property="og:image" content="https://web-dashers.github.io/wd_assets/thumbnail_1024.png" />

    <meta property="twitter:card" content="summary_large_image" />
    <meta property="twitter:url" content="https://web-dashers.github.io/" />
    <meta property="twitter:title" content="Web Dashers" />
    <meta property="twitter:description" content="Web Dashers is a fan-made browser mod of the Geometry Dash Web Demo. It features the main levels, a built-in level creator, and a level search to play online levels. The project is actively in development, with more features and updates planned." />
    <meta property="twitter:image" content="https://web-dashers.github.io/wd_assets/thumbnail_1024.png" />

    <script src="assets/scripts/game/allObjects.js?latest"></script>
    <script src="assets/scripts/game/allLevels.js"></script>
    <script src="assets/scripts/libs/phaser.min.js"></script>
    <script src="assets/scripts/libs/pako.min.js"></script>
    <script src="assets/scripts/utils/cache-manager.js"></script>
    <script src="assets/scripts/api/GDapiWrapper.js"></script>
    <script src="assets/scripts/utils/api-config.js"></script>
    <script src="assets/scripts/api/account-api.js"></script>

    <script>
        (function () {
            var GD_PROXY_HOSTS = [
                "https://gd-proxy.webdashers.workers.dev",
                "https://webdashers.webdashersdevelopement.workers.dev/",
                "fallbackgdproxy.webdashers.workers.dev",
                "fallback2.webdashers.workers.dev"
            ];

            window._gdProxyUrl = GD_PROXY_HOSTS[0];
            window._gdProxyHosts = GD_PROXY_HOSTS;

            var activeIndex = 0;
            var nativeFetch = window.fetch.bind(window);

            function matchingHost(url) {
                for (var i = 0; i < GD_PROXY_HOSTS.length; i++) {
                    if (url.indexOf(GD_PROXY_HOSTS[i]) === 0) return GD_PROXY_HOSTS[i];
                }
                return null;
            }

            function withHost(url, oldHost, newHost) {
                return newHost + url.slice(oldHost.length);
            }

            window.fetch = function (input, init) {
                var isRequestObj = (typeof Request !== "undefined") && (input instanceof Request);
                var url = isRequestObj ? input.url : input;

                if (typeof url !== "string") {
                    return nativeFetch(input, init);
                }

                var oldHost = matchingHost(url);
                if (!oldHost) {
                    return nativeFetch(input, init);
                }

                var attempts = 0;
                var lastError = null;

                function tryHost(hostIndex) {
                    var host = GD_PROXY_HOSTS[hostIndex];
                    var attemptUrl = withHost(url, oldHost, host);
                    var attemptInput = isRequestObj ? new Request(attemptUrl, input) : attemptUrl;

                    return nativeFetch(attemptInput, init).then(function (res) {
                        if ((res.status === 429 || res.status === 503) && attempts < GD_PROXY_HOSTS.length - 1) {
                            attempts++;
                            return tryHost((hostIndex + 1) % GD_PROXY_HOSTS.length);
                        }
                        activeIndex = hostIndex;
                        window._gdProxyUrl = host;
                        return res;
                    }, function (err) {
                        lastError = err;
                        if (attempts < GD_PROXY_HOSTS.length - 1) {
                            attempts++;
                            return tryHost((hostIndex + 1) % GD_PROXY_HOSTS.length);
                        }
                        throw lastError;
                    });
                }

                return tryHost(activeIndex);
            };
        })();
    </script>

    <script defer src="assets/scripts/utils/config.js"></script>
    <script defer src="assets/scripts/core/triggers.js"></script>
    <script defer src="assets/scripts/core/level.js?latest"></script>
    <script defer src="assets/scripts/core/player.js?latehkjthykhst"></script>
    <script defer src="assets/scripts/core/audio.js"></script>
    <script defer src="assets/scripts/core/loading-screen.js"></script>
    <script defer src="assets/scripts/core/level-editor.js?latest"></script>
    <script defer src="assets/scripts/core/game-scene.js?latest3"></script>
    <script defer src="assets/scripts/core/main.js"></script>

    <style>
        html, body {
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0;
            overflow: hidden;
            background: #000;
        }

        body {
            position: relative;
        }

        #wd-mod-button {
            position: fixed;
            top: 12px;
            right: 12px;
            z-index: 99999;
            padding: 10px 14px;
            border: 0;
            border-radius: 10px;
            background: rgba(0, 0, 0, 0.75);
            color: #fff;
            font-family: Arial, sans-serif;
            font-size: 14px;
            cursor: pointer;
        }

        #wd-mod-panel {
            position: fixed;
            top: 60px;
            right: 12px;
            width: min(340px, calc(100vw - 24px));
            max-height: calc(100vh - 80px);
            overflow: auto;
            z-index: 99999;
            display: none;
            background: rgba(18, 18, 18, 0.96);
            color: #fff;
            border-radius: 14px;
            padding: 16px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.35);
            font-family: Arial, sans-serif;
        }

        #wd-mod-panel.open {
            display: block;
        }

        #wd-mod-panel h2 {
            margin: 0 0 12px;
            font-size: 18px;
        }

        #wd-mod-panel .wd-section {
            margin-bottom: 14px;
            padding-bottom: 12px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.12);
        }

        #wd-mod-panel .wd-section:last-child {
            border-bottom: none;
            padding-bottom: 0;
            margin-bottom: 0;
        }

        #wd-mod-panel label {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 12px;
            margin: 8px 0;
            font-size: 14px;
        }

        #wd-mod-panel input[type="checkbox"] {
            transform: scale(1.15);
        }

        #wd-mod-panel select,
        #wd-mod-panel input[type="number"] {
            width: 100%;
            margin-top: 6px;
            padding: 8px 10px;
            border-radius: 8px;
            border: 1px solid rgba(255, 255, 255, 0.16);
            background: rgba(255, 255, 255, 0.08);
            color: #fff;
            outline: none;
        }

        #wd-mod-panel button {
            width: 100%;
            margin-top: 8px;
            padding: 10px 12px;
            border: 0;
            border-radius: 10px;
            background: #c1fb3d;
            color: #111;
            font-weight: bold;
            cursor: pointer;
        }

        #wd-mod-panel .wd-small {
            font-size: 12px;
            opacity: 0.8;
            line-height: 1.35;
        }

        #wd-mod-close {
            float: right;
            background: transparent !important;
            color: #fff !important;
            width: auto !important;
            padding: 0 !important;
            margin: 0 !important;
            font-size: 18px;
        }
    </style>

    <script>
        (function () {
            var STORAGE_KEY = "wd_mod_config_v1";

            var defaultConfig = {
                godMode: false,
                noClip: false,
                practiceMode: false,
                autoRestart: false,
                showHitboxes: false,
                speedMultiplier: 1,
                texturePack: "default"
            };

            function loadConfig() {
                try {
                    var saved = JSON.parse(localStorage.getItem(STORAGE_KEY));
                    return Object.assign({}, defaultConfig, saved || {});
                } catch (e) {
                    return Object.assign({}, defaultConfig);
                }
            }

            function saveConfig(cfg) {
                try {
                    localStorage.setItem(STORAGE_KEY, JSON.stringify(cfg));
                } catch (e) {}
            }

            function setTexturePack(pack) {
                var existing = document.getElementById("wd-texture-pack-stylesheet");

                if (!existing) {
                    existing = document.createElement("link");
                    existing.id = "wd-texture-pack-stylesheet";
                    existing.rel = "stylesheet";
                    document.head.appendChild(existing);
                }

                if (pack && pack !== "default") {
                    existing.href = "assets/texture-packs/" + encodeURIComponent(pack) + "/style.css";
                } else {
                    existing.removeAttribute("href");
                }
            }

            function applyConfig(cfg) {
                window.WD_MOD_CONFIG = cfg;
                window.WD_CHEATS = cfg;
                window.WD_TEXTURE_PACK = cfg.texturePack;
                setTexturePack(cfg.texturePack);
            }

            function buildMenu() {
                var button = document.createElement("button");
                button.id = "wd-mod-button";
                button.textContent = "Mods";

                var panel = document.createElement("div");
                panel.id = "wd-mod-panel";
                panel.innerHTML = `
                    <button id="wd-mod-close" aria-label="Close menu">✕</button>
                    <h2>Mod / Cheat Menu</h2>

                    <div class="wd-section">
                        <label><span>God Mode</span><input type="checkbox" id="wd-godmode"></label>
                        <label><span>No Clip</span><input type="checkbox" id="wd-noclip"></label>
                        <label><span>Practice Mode</span><input type="checkbox" id="wd-practice"></label>
                        <label><span>Auto Restart</span><input type="checkbox" id="wd-autorestart"></label>
                        <label><span>Show Hitboxes</span><input type="checkbox" id="wd-hitboxes"></label>
                    </div>

                    <div class="wd-section">
                        <label><span>Speed Multiplier</span></label>
                        <input type="number" id="wd-speed" min="0.25" max="4" step="0.05">
                        <div class="wd-small">Example: 1 = normal speed, 2 = double speed.</div>
                    </div>

                    <div class="wd-section">
                        <label><span>Texture Pack</span></label>
                        <select id="wd-texture-pack">
                            <option value="default">Default</option>
                            <option value="classic">Classic</option>
                            <option value="neon">Neon</option>
                            <option value="pixel">Pixel</option>
                        </select>
                        <div class="wd-small">Put pack CSS files in <code>assets/texture-packs/&lt;pack&gt;/style.css</code>.</div>
                    </div>

                    <div class="wd-section">
                        <button id="wd-reset">Reset to Defaults</button>
                    </div>

                    <div class="wd-small">
                        Open/close menu: <b>M</b><br>
                        Press <b>Esc</b> to close.
                    </div>
                `;

                document.body.appendChild(button);
                document.body.appendChild(panel);

                var cfg = loadConfig();
                applyConfig(cfg);

                function syncInputs() {
                    document.getElementById("wd-godmode").checked = cfg.godMode;
                    document.getElementById("wd-noclip").checked = cfg.noClip;
                    document.getElementById("wd-practice").checked = cfg.practiceMode;
                    document.getElementById("wd-autorestart").checked = cfg.autoRestart;
                    document.getElementById("wd-hitboxes").checked = cfg.showHitboxes;
                    document.getElementById("wd-speed").value = cfg.speedMultiplier;
                    document.getElementById("wd-texture-pack").value = cfg.texturePack;
                }

                function persistAndApply() {
                    saveConfig(cfg);
                    applyConfig(cfg);
                }

                function togglePanel(force) {
                    var open = typeof force === "boolean" ? force : !panel.classList.contains("open");
                    panel.classList.toggle("open", open);
                }

                button.addEventListener("click", function () {
                    togglePanel();
                });

                document.getElementById("wd-mod-close").addEventListener("click", function () {
                    togglePanel(false);
                });

                document.getElementById("wd-reset").addEventListener("click", function () {
                    cfg = Object.assign({}, defaultConfig);
                    syncInputs();
                    persistAndApply();
                });

                document.getElementById("wd-godmode").addEventListener("change", function (e) {
                    cfg.godMode = e.target.checked;
                    persistAndApply();
                });

                document.getElementById("wd-noclip").addEventListener("change", function (e) {
                    cfg.noClip = e.target.checked;
                    persistAndApply();
                });

                document.getElementById("wd-practice").addEventListener("change", function (e) {
                    cfg.practiceMode = e.target.checked;
                    persistAndApply();
                });

                document.getElementById("wd-autorestart").addEventListener("change", function (e) {
                    cfg.autoRestart = e.target.checked;
                    persistAndApply();
                });

                document.getElementById("wd-hitboxes").addEventListener("change", function (e) {
                    cfg.showHitboxes = e.target.checked;
                    persistAndApply();
                });

                document.getElementById("wd-speed").addEventListener("change", function (e) {
                    var value = parseFloat(e.target.value);
                    if (isNaN(value)) value = 1;
                    cfg.speedMultiplier = Math.max(0.25, Math.min(4, value));
                    e.target.value = cfg.speedMultiplier;
                    persistAndApply();
                });

                document.getElementById("wd-texture-pack").addEventListener("change", function (e) {
                    cfg.texturePack = e.target.value;
                    persistAndApply();
                });

                syncInputs();

                window.addEventListener("keydown", function (e) {
                    if (e.key === "m" || e.key === "M") togglePanel();
                    if (e.key === "Escape") togglePanel(false);
                });
            }

            function start() {
                try {
                    buildMenu();
                } catch (err) {
                    console.error("Mod menu error:", err);
                }
            }

            if (document.readyState === "loading") {
                document.addEventListener("DOMContentLoaded", start);
            } else {
                start();
            }
        })();
    </script>

    <script>
        document.addEventListener("contextmenu", function (event) {
            event.preventDefault();
        });
    </script>
</head>

<body>
</body>
</html>
