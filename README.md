-- Dark Max Anti-Lag Brookhaven V4 | COMPLETE & Delta-Compatible 2026

if getgenv().DarkMaxLoaded then 
    print("Already loaded - skipping")
    return 
end
getgenv().DarkMaxLoaded = true

print("Dark Max V4 starting...")

-- Try load Kavo (primary + backup)
local Library = nil
local success, err = pcall(function()
    Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua", true))()
end)

if not success then
    warn("Primary Kavo failed: " .. tostring(err))
    success, err = pcall(function()
        Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/bloodball/-back-ups-for-libs/main/kavo", true))()
    end)
end

if not success or not Library then
    warn("Kavo completely failed. Script will stop. Try manual load or different executor.")
    return
end

local Window = Library.CreateLib("Dark Max Brookhaven | 2026", "DarkTheme")

local Main = Window:NewTab("Main")
local Troll = Window:NewTab("Troll")
local AntiLagTab = Window:NewTab("Anti-Lag")

local MainSection = Main:NewSection("Player Stuff")
local TrollSection = Troll:NewSection("Troll Stuff")
local AntiSection = AntiLagTab:NewSection("Lag Killer")

-- FPS Display
local fpsLbl = AntiSection:NewLabel("FPS: ...")
task.spawn(function()
    while getgenv().DarkMaxLoaded do
        local fps = math.floor(1 / game:GetService("RunService").RenderStepped:Wait())
        fpsLbl:Set("FPS: " .. fps)
        task.wait(1)
    end
end)

-- Anti-Lag Core
local lagCount = 0
local lastCleanTime = tick()
local function cleanLagSources()
    local now = tick()
    if now - lastCleanTime < 0.5 then return end
    lastCleanTime = now
    
    pcall(function()
        lagCount = 0
        for _, v in ipairs(workspace:GetDescendants()) do
            if v:IsA("Explosion") or v:IsA("ParticleEmitter") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") or
               (v:IsA("BasePart") and (v.Name:lower():find("bomb") or v.Name:lower():find("explos"))) then
                lagCount += 1
                v:Destroy()
            end
        end
        if lagCount > 4 then
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "Anti-Lag", Text = "Removed " .. lagCount .. " lag sources", Duration = 3
            })
        end
    end)
end

task.spawn(function()
    while getgenv().DarkMaxLoaded do
        cleanLagSources()
        task.wait(0.4)
    end
end)

-- Hook to block spam remotes + kick
local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
    local m = getnamecallmethod()
    if m == "FireServer" or m == "InvokeServer" then
        local name = tostring(self):lower()
        if name:find("expl") or name:find("bomb") or name:find("spam") or name:find("nuke") then
            lagCount += 1
            if lagCount > 5 then return end
        end
    end
    if m == "Kick" then
        warn("Kick blocked!")
        return
    end
    return old(self, ...)
end)

setreadonly(mt, true)

-- Performance
settings().Rendering.QualityLevel = "Level01"
settings().Physics.AllowSleep = true

AntiSection:NewLabel("Anti-Lag: ON (clean + block)")
AntiSection:NewButton("Clean Now", cleanLagSources)

-- Fly
local flySpeed = 50
local flying = false
local flyConn

local function startFly(state)
    local plr = game.Players.LocalPlayer
    local char = plr.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    local root = char.HumanoidRootPart
    flying = state

    if state then
        local bg = Instance.new("BodyGyro", root)
        bg.MaxTorque = Vector3.new(1e9,1e9,1e9)
        bg.P = 9e4

        local bv = Instance.new("BodyVelocity", root)
        bv.MaxForce = Vector3.new(1e9,1e9,1e9)

        flyConn = game:GetService("RunService").RenderStepped:Connect(function()
            if not flying then return end
            local cam = workspace.CurrentCamera
            local dir = Vector3.zero
            local uis = game:GetService("UserInputService")

            if uis:IsKeyDown(Enum.KeyCode.W) then dir += cam.CFrame.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.S) then dir -= cam.CFrame.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.A) then dir -= cam.CFrame.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.D) then dir += cam.CFrame.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.yAxis end
            if uis:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.yAxis end

            bv.Velocity = dir.Unit * flySpeed * (uis:IsKeyDown(Enum.KeyCode.LeftShift) and 2 or 1)
            bg.CFrame = cam.CFrame
        end)
    else
        if flyConn then flyConn:Disconnect() end
        if root:FindFirstChild("BodyVelocity") then root.BodyVelocity:Destroy() end
        if root:FindFirstChild("BodyGyro") then root.BodyGyro:Destroy() end
    end
end

MainSection:NewToggle("Fly (WASD + Space/Ctrl)", startFly)
MainSection:NewSlider("Fly Speed", 10, 300, function(v) flySpeed = v end)

-- WalkSpeed
MainSection:NewSlider("WalkSpeed", 16, 500, function(v)
    local h = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
    if h then h.WalkSpeed = v end
end)

-- Invisible Toggle
MainSection:NewButton("Toggle Invisible", function()
    local char = game.Players.LocalPlayer.Character
    if not char then return end
    local invisible = false
    for _, p in char:GetDescendants() do
        if p:IsA("BasePart") or p:IsA("MeshPart") then
            if p.Transparency < 0.9 then
                p.Transparency = 1
                invisible = true
            else
                p.Transparency = 0
            end
        end
    end
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Dark Max", Text = invisible and "Invisible!" or "Visible!", Duration = 3
    })
end)

-- Bring All
TrollSection:NewButton("Bring All Players", function()
    local me = game.Players.LocalPlayer.Character
    if not me or not me:FindFirstChild("HumanoidRootPart") then return end
    local pos = me.HumanoidRootPart.Position
    local cnt = 0
    for _, p in game.Players:GetPlayers() do
        if p \~= game.Players.LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            pcall(function()
                p.Character.HumanoidRootPart.CFrame = CFrame.new(pos + Vector3.new(math.random(-10,10), 4, math.random(-10,10)))
                cnt += 1
            end)
        end
    end
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Troll", Text = "Brought " .. cnt .. " players!", Duration = 4
    })
end)

print("Dark Max V4 Loaded - All features ready")
game:GetService("StarterGui"):SetCore("SendNotification", {Title = "Dark Max", Text = "Loaded! Anti-Lag Active", Duration = 5})
