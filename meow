-- Farming Automation Script with Tweening & UI
-- Matcha LuaVM Roblox Executor

-- ─── Load library ────────────────────────────────────────────
local Library
do
    local ok, err = pcall(function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/myth4c/Scripts/refs/heads/main/wabisabi%20lib"))()
        Library = WabiSabiUILibrary
    end)
    if not ok then
        warn("Failed to load WabiSabi UI: " .. tostring(err))
        return
    end
    if not Library then
        warn("WabiSabiUILibrary not found after load")
        return
    end
    print("✓ WabiSabiUILibrary loaded successfully")
end

local RunService = game:GetService("RunService")
local runService = RunService
local player = game:GetService("Players").LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- ==================== CONFIGURATION ====================
local JOJOS_FARMING_SPOT = Vector3.new(-3077.804931640625, 7.518670558929443, -664.4952392578125)
local HUNT_LOCATION = Vector3.new(-1822.32421875, 2.667064666748047, 1901.2093505859375)
local TWEEN_SPEED = 50 -- Speed in studs per second
local HUNT_INTERVAL = 15 * 60 -- 15 minutes between hunts

-- ==================== STATE MANAGEMENT ====================
local state = {
  scriptEnabled = false,
  lastHuntTime = 0,
  tweenActive = false
}

local tweenConn = nil
local initialActivation = false
local onHunt = false

-- ==================== NPC DETECTION ====================
local function checkForNPC()
  local npcFolder = game.Workspace:FindFirstChild("NPCs")
  if npcFolder then
    local boss = npcFolder:FindFirstChild("CosmicBeingBoss_Normal")
    if boss then
      return true
    end
  end
  return false
end

local function getBossPosition()
  local npcFolder = game.Workspace:FindFirstChild("NPCs")
  if npcFolder then
    local boss = npcFolder:FindFirstChild("CosmicBeingBoss_Normal")
    if boss then
      local root = boss:FindFirstChild("HumanoidRootPart") or boss:FindFirstChildOfClass("BasePart")
      if root then
        return root.Position
      end
    end
  end
  return nil
end

local function cancelTween()
    state.tweenActive = false
    if tweenConn then tweenConn:Disconnect(); tweenConn = nil end
    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then pcall(function() hrp.AssemblyLinearVelocity = Vector3.zero end) end
end

local function tweenToTarget(targetPos)
    if state.tweenActive then cancelTween() end
    state.tweenActive = true
    tweenConn = RunService.Heartbeat:Connect(function()
        if not state.tweenActive then
            if tweenConn then tweenConn:Disconnect(); tweenConn = nil end
            return
        end
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then cancelTween(); return end
        local diff = targetPos - hrp.Position
        if diff.Magnitude < 10 then
            pcall(function() hrp.AssemblyLinearVelocity = Vector3.zero end)
            cancelTween()
            return
        end
        local speed = TWEEN_SPEED
        local dir = diff.Unit
        pcall(function()
            hrp.AssemblyLinearVelocity = Vector3.new(
                dir.X * speed,
                math.max(dir.Y * speed * 0.5, 0),
                dir.Z * speed
            )
        end)
    end)
end

-- ==================== RETURN TO FARM SPOT ====================
local function returnToFarmSpot()
  if not state.scriptEnabled then return end
  local distance = (JOJOS_FARMING_SPOT - humanoidRootPart.Position).Magnitude
  print("Returning to Jojos Farming Spot (Distance: " .. math.floor(distance) .. " studs)")
  tweenToTarget(JOJOS_FARMING_SPOT)
  
  -- Wait for tween to complete
  while state.tweenActive and state.scriptEnabled do
    task.wait(0.1)
  end
end

-- ==================== UI WINDOW ====================
local Window
local createWindowOk, createWindowErr = pcall(function()
    Window = Library:CreateWindow({
        Title   = "Farming Automation",
        Columns = 1,
        Divider = true,
        ConfigFile = "farming_automation.json",
        BuiltInIndicatorToggle = false,
        Width   = 300,
        Theme   = { Colors = { Accent = Color3.fromRGB(0, 170, 220) } }
    })
end)

if not createWindowOk or not Window then
    warn("Failed to create window: " .. tostring(createWindowErr))
    print("CreateWindow returned:", Window)
    return
end

print("✓ Window created successfully")

task.spawn(function()
    task.wait(0.2)
    pcall(function()
        Window.Indicator:SetVisible(false)
        Window.Config.IndicatorEnabled = false
    end)
end)

Window:AddSection("Automation", { Column = 1 })

Window:AddToggle("script_enabled", {
    Text        = "Enable Automation",
    Description = "Farm at Jojos, hunt every 15 mins",
    Default     = false,
    Keybind     = true,
    Column      = 1,
    Callback    = function(v)
        state.scriptEnabled = v
        if v then
            initialActivation = true  -- Mark that we just turned on
            state.lastHuntTime = tick()  -- Reset hunt timer
        end
        print("Automation " .. (v and "ON!" or "OFF!"))
    end
})

Window:AddToggle("noclip_enabled", {
    Text        = "Noclip",
    Description = "Disable collision on character parts",
    Default     = false,
    Column      = 1,
    Callback    = function(v)
        if v then
            _G.noclipConn = runService.Stepped:Connect(function()
                local character = player.Character
                if not character then return end
                for _, part in ipairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end)
            print("Noclip ON")
        else
            if _G.noclipConn then
                _G.noclipConn:Disconnect()
                _G.noclipConn = nil
            end
            print("Noclip OFF")
        end
    end
})

Window:AddSection("Boss", { Column = 1 })

Window:AddButton("tween_to_boss", {
    Text    = "Tween to Boss",
    Column  = 1,
    Callback = function()
        local bossPos = getBossPosition()
        if bossPos then
            print("Tweening to Boss...")
            tweenToTarget(bossPos)
        else
            print("Boss not found!")
        end
    end
})

Window:AddSection("Settings", { Column = 1 })

Window:AddSlider("tween_speed", {
    Text        = "Tween Speed",
    Description = "Speed in studs per second",
    Min         = 10,
    Max         = 200,
    Default     = 50,
    Rounding    = 5,
    Column      = 1,
    Callback    = function(v) 
        TWEEN_SPEED = v
    end
})

print("✓ UI elements added successfully")

-- ==================== MAIN LOOP ====================
local bossTweened = false
local function mainLoop()
  while true do
    task.wait(1)

    if state.scriptEnabled then
      -- On initial activation, tween to hunt location and search for boss
      if initialActivation then
        print("Initial activation: Tweening to hunt location...")
        tweenToTarget(HUNT_LOCATION)
        while state.tweenActive do
          task.wait(0.1)
        end
        tweenToTarget(Vector3.new(HUNT_LOCATION.X, HUNT_LOCATION.Y + 40, HUNT_LOCATION.Z))
        while state.tweenActive do
          task.wait(0.1)
        end
        
        print("[HUNT] Searching for boss...")
        if checkForNPC() then
          local bossPos = getBossPosition()
          if bossPos then
            print("[HUNT] Boss found! Tweening to boss...")
            tweenToTarget(bossPos)
            while state.tweenActive do
              task.wait(0.1)
            end
            tweenToTarget(Vector3.new(bossPos.X, bossPos.Y + 40, bossPos.Z))
            while state.tweenActive do
              task.wait(0.1)
            end
            
            -- Follow boss until dead
            print("[HUNT] Following boss...")
            while checkForNPC() and state.scriptEnabled do
              local currentBossPos = getBossPosition()
              if currentBossPos then
                local distance = (currentBossPos - player.Character.HumanoidRootPart.Position).Magnitude
                if distance > 50 then
                  tweenToTarget(currentBossPos)
                end
              end
              task.wait(0.5)
            end
            
            -- FIX: Cancel any in-progress tween before returning
            print("[HUNT] Boss defeated/lost. Returning to Jojos...")
            cancelTween()
            task.wait(0.2)
            tweenToTarget(JOJOS_FARMING_SPOT)
            while state.tweenActive and state.scriptEnabled do
              task.wait(0.1)
            end
            tweenToTarget(Vector3.new(JOJOS_FARMING_SPOT.X, JOJOS_FARMING_SPOT.Y + 40, JOJOS_FARMING_SPOT.Z))
            while state.tweenActive and state.scriptEnabled do
              task.wait(0.1)
            end
            print("[HUNT] Arrived at Jojos.")
          end
        else
          print("No boss found at hunt location.")
        end
        
        -- Always return to Jojos after initial hunt (covers no-boss case)
        cancelTween()
        task.wait(0.2)
        tweenToTarget(JOJOS_FARMING_SPOT)
        while state.tweenActive do
          task.wait(0.1)
        end
        tweenToTarget(Vector3.new(JOJOS_FARMING_SPOT.X, JOJOS_FARMING_SPOT.Y + 40, JOJOS_FARMING_SPOT.Z))
        while state.tweenActive do
          task.wait(0.1)
        end
        
        initialActivation = false
        state.lastHuntTime = tick()
      end

      local timeSinceHunt = tick() - state.lastHuntTime
      
      -- Check if it's time for a hunt (every 15 minutes)
      if timeSinceHunt >= HUNT_INTERVAL and not onHunt then
        print("[HUNT CYCLE] Starting hunt...")
        onHunt = true
        bossTweened = false
        
        -- Tween to hunt location
        print("Tweening to hunt location...")
        tweenToTarget(HUNT_LOCATION)
        while state.tweenActive do
          task.wait(0.1)
        end
        tweenToTarget(Vector3.new(HUNT_LOCATION.X, HUNT_LOCATION.Y + 40, HUNT_LOCATION.Z))
        while state.tweenActive do
          task.wait(0.1)
        end
        
        -- Search for boss at hunt location
        print("[HUNT] Searching for boss at hunt location...")
        local huntStartTime = tick()
        local maxHuntDuration = 5 * 60 -- 5 minute max hunt
        
        while tick() - huntStartTime < maxHuntDuration and state.scriptEnabled and onHunt do
          if checkForNPC() then
            local bossPos = getBossPosition()
            if bossPos then
              print("[HUNT] Boss found! Tweening to boss...")
              tweenToTarget(bossPos)
              bossTweened = true
              
              -- Wait for tween to complete
              while state.tweenActive and state.scriptEnabled do
                task.wait(0.1)
              end
              tweenToTarget(Vector3.new(bossPos.X, bossPos.Y + 40, bossPos.Z))
              while state.tweenActive and state.scriptEnabled do
                task.wait(0.1)
              end
              
              -- Follow boss until dead
              while checkForNPC() and state.scriptEnabled and onHunt do
                local currentBossPos = getBossPosition()
                if currentBossPos then
                  local distance = (currentBossPos - player.Character.HumanoidRootPart.Position).Magnitude
                  if distance > 50 then
                    tweenToTarget(currentBossPos)
                  end
                end
                task.wait(0.5)
              end
              
              -- FIX: Cancel any in-progress tween before returning
              print("[HUNT] Boss defeated/lost. Returning to Jojos...")
              cancelTween()
              task.wait(0.2)
              tweenToTarget(JOJOS_FARMING_SPOT)
              while state.tweenActive and state.scriptEnabled do
                task.wait(0.1)
              end
              print("[HUNT] Arrived at Jojos.")
              break
            end
          end
          task.wait(1)
        end
        
        -- Return to Jojos farm (covers no-boss / timeout case)
        print("Hunt cycle complete. Returning to Jojos Farm Spot...")
        cancelTween()
        task.wait(0.2)
        tweenToTarget(JOJOS_FARMING_SPOT)
        while state.tweenActive do
          task.wait(0.1)
        end
        
        -- Reset hunt timer
        state.lastHuntTime = tick()
        onHunt = false
        bossTweened = false
        print("[CYCLE] Waiting " .. HUNT_INTERVAL / 60 .. " minutes before next hunt...")
      end
    end
  end
end

-- ==================== INPUT HANDLING ====================
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F5 then
        state.scriptEnabled = not state.scriptEnabled
        print("Script toggled via F5: " .. (state.scriptEnabled and "ON" or "OFF"))
    end
end)

-- ==================== INITIALIZATION ====================
print("=== FARMING AUTOMATION SCRIPT LOADED ===")
print("Press F5 to toggle script on/off or use UI toggle")
print("Farms at Jojos spot. Every 15 mins: travels to hunt location, searches for boss, and follows if found.")
print("When hunt complete or boss dies: Returns to Jojos and waits for next hunt cycle.")

-- Start main automation loop
task.spawn(mainLoop)

-- Main loop with Window:Step()
print("✓ Starting UI loop...")
while true do
    local ok, err = pcall(function() 
        if Window and Window.Step then
            Window:Step() 
        end
    end)  
    if not ok then 
        warn("Step error: " .. tostring(err))
        task.wait(1) 
    end
    task.wait()
end
