-- ==============================================================
-- TKSH v35 — Script completo corregido (final arreglado)
-- Tema: Amarillo Pastel | UI: WindUI
-- ==============================================================

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerName, DisplayName, UserId = LocalPlayer.Name, LocalPlayer.DisplayName, LocalPlayer.UserId
AntiAFKEnabled, NoFrictionEnabled, NoPushEnabled, NoKnockbackEnabled = false, false, false, false
FlyEnabled, FlySpeed = false, 60
FlyConn, FlyVel, FlyGyro, NoclipEnabled, NoclipConn = nil, nil, nil, false, nil
ESPEnabled, ESPConn, FullbrightEnabled, AutoRejoinEnabled = false, nil, false, false
AntiVoidEnabled, AntiRagdollEnabled, InfJumpEnabled, InfJumpConn = false, false, false, nil
FpsBoostEnabled = false
LastSafePos = Vector3.new(0,10,0)
SpawnCFrame = CFrame.new()
SpectateName, TPPlayerName, SpectateDropdown, TPPlayerDropdown, CoordsParagraph = nil, nil, nil, nil, nil
GodModeEnabled, ClickTPEnabled, AutoJumpEnabled, WalkOnWaterEnabled = false, false, false, false
FollowPlayerEnabled, SpinBotEnabled, FreezePositionEnabled, InstantRespawnEnabled = false, false, false, false
FallSpeedCap, SavedPosition, CoordsText, ClickTPConn, SpinConn = 200, nil, "", nil, nil
AutoWalkEnabled, AutoWalkConn = false, nil
TPWalkEnabled, TPWalkSpeed, TPWalkConn = false, 0.30, nil
InvisibleEnabled = false
HidePosEnabled, HidePosConn, HideParts = false, nil, {}
AutoClickerEnabled, AutoClickerCPS = false, 10
NoFallDamageEnabled, NoFallConn = false, nil
AntiAFKConn = nil
local Lighting = game:GetService("Lighting")
local OriginalLight = {ClockTime=Lighting.ClockTime, Brightness=Lighting.Brightness, Ambient=Lighting.Ambient, OutdoorAmbient=Lighting.OutdoorAmbient, FogEnd=Lighting.FogEnd, GlobalShadows=Lighting.GlobalShadows, ExposureCompensation=Lighting.ExposureCompensation}
local GameSettings = UserSettings().GameSettings
local OriginalQuality = Enum.SavedQualitySetting.Automatic
pcall(function() OriginalQuality = GameSettings.SavedQualityLevel end)

-- ⚔️ ATAQUE RÁPIDO
local FlashAttackEnabled = false
local FlashMultiplier = 5
local FlashConn = nil

-- 🛡️ SIT PROTECTOR
local SitProtectorEnabled = false
local SitConexiones = {}
local SitEstadosOriginales = {}
local SitGui = nil

-- 🛡️ ESCUDOS REALES
local AntiKickEnabled, AntiResetEnabled, AntiSitEnabled = false, false, false
local AntiFlingEnabled, AntiFreezeEnabled, AntiReportEnabled = false, false, false
local AntiKickHooked = false
local OldNameCall = nil

-- 🎙️ ANTI-VC ULTRA
local AntiVCEnabled = false
local AntiVCConn, AntiVCFastConn = nil, nil
local VoiceChatService = nil
pcall(function() VoiceChatService = game:GetService("VoiceChatService") end)

-- 🎯 MODULO TARGET
local TargetToggles = {Fling=false, View=false, Focus=false, Bang=false, HeadSit=false, Stand=false, Backpack=false, Doggy=false, Drag=false}
local TargetedPlayerName = nil
local TargetWhitelist = {}
local TargetLoopConn = nil
TargetDropdown, TargetInfoParagraph, TargetAvatarImg, TargetNameInput = nil, nil, nil, nil

local Character, Humanoid, RootPart
local function UpdateChar()
    Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    Humanoid = Character:WaitForChild("Humanoid")
    RootPart = Character:WaitForChild("HumanoidRootPart")
    SpawnCFrame = RootPart.CFrame
end
UpdateChar()
LocalPlayer.CharacterAdded:Connect(UpdateChar)

local function GetPlayerNames(includeNone)
    local names = {}
    if includeNone ~= false then table.insert(names, "Nadie (detener)") end
    for _, p in ipairs(Players:GetPlayers()) do if p ~= LocalPlayer then table.insert(names, p.Name) end end
    if #names == 0 then table.insert(names, "No hay jugadores") end
    return names
end
local function TeleportToCoords(text)
    if not text or text == "" then WindUI:Notify({Title="Error", Content="Texto vacío", Duration=2}) return end
    local x,y,z = text:match("([%-%d%.]+)[,%s]+([%-%d%.]+)[,%s]+([%-%d%.]+)")
    x,y,z = tonumber(x), tonumber(y), tonumber(z)
    if x and y and z and RootPart then
        RootPart.CFrame = CFrame.new(x,y,z); RootPart.Velocity = Vector3.new(0,0,0)
        WindUI:Notify({Title="TP", Content="X:"..x.." Y:"..y.." Z:"..z, Duration=2})
    else WindUI:Notify({Title="Error", Content="Formato: X, Y, Z", Duration=3}) end
end

-- FLY
local function EnsureFlyMovers()
    if not RootPart then return false end
    if not RootPart:FindFirstChild("WindFlyVel") then
        local bv = Instance.new("BodyVelocity"); bv.Name="WindFlyVel"; bv.MaxForce=Vector3.new(9e9,9e9,9e9); bv.Velocity=Vector3.new(0,0,0); bv.Parent=RootPart; FlyVel=bv
    else FlyVel = RootPart.WindFlyVel end
    if not RootPart:FindFirstChild("WindFlyGyro") then
        local bg = Instance.new("BodyGyro"); bg.Name="WindFlyGyro"; bg.P=9e4; bg.MaxTorque=Vector3.new(9e9,9e9,9e9); bg.Parent=RootPart; FlyGyro=bg
    else FlyGyro = RootPart.WindFlyGyro end
    if Humanoid then Humanoid.PlatformStand=true; Humanoid.GravityScale=0 end
    return true
end
local function StartFly()
    FlyEnabled=true
    if not FlyConn then
        FlyConn = RunService.RenderStepped:Connect(function()
            if not FlyEnabled or not EnsureFlyMovers() then return end
            local cf = workspace.CurrentCamera.CFrame
            local move = Vector3.new(0,0,0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then move=move+cf.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then move=move-cf.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then move=move-cf.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then move=move+cf.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move=move+Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then move=move-Vector3.new(0,1,0) end
            if move.Magnitude>0 then move=move.Unit end
            FlyVel.Velocity = move*FlySpeed; FlyGyro.CFrame=cf
        end)
    end
end
local function StopFly()
    FlyEnabled=false
    if FlyConn then FlyConn:Disconnect(); FlyConn=nil end
    if RootPart then
        local bv=RootPart:FindFirstChild("WindFlyVel"); if bv then bv:Destroy() end
        local bg=RootPart:FindFirstChild("WindFlyGyro"); if bg then bg:Destroy() end
    end
    if Humanoid then Humanoid.PlatformStand=false; Humanoid.GravityScale=1 end
end
local function SetNoclip(state)
    NoclipEnabled=state
    if state then
        if not NoclipConn then NoclipConn=RunService.Stepped:Connect(function()
            if not NoclipEnabled or not Character then return end
            for _,v in ipairs(Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide=false end end
        end) end
    else if NoclipConn then NoclipConn:Disconnect(); NoclipConn=nil end end
end
local function SetInfJump(state)
    InfJumpEnabled=state
    if state then
        if not InfJumpConn then InfJumpConn=UserInputService.JumpRequest:Connect(function()
            if Humanoid then Humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
        end) end
    else if InfJumpConn then InfJumpConn:Disconnect(); InfJumpConn=nil end end
end

-- TPWALK
getgenv().TPWalkEnabled = false
getgenv().TPWalkSpeed = TPWalkSpeed
local function SetTPWalk(s)
    TPWalkEnabled = s
    getgenv().TPWalkEnabled = s
    if s then
        if not TPWalkConn then
            TPWalkConn = RunService.RenderStepped:Connect(function()
                if not getgenv().TPWalkEnabled then return end
                local character = LocalPlayer.Character
                if not character then return end
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                local hrp = character:FindFirstChild("HumanoidRootPart")
                local camera = workspace.CurrentCamera
                if not humanoid or not hrp or not camera then return end
                humanoid.Sit = true
                humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
                humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                humanoid:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, false)
                humanoid.PlatformStand = false
                local shiftLockActivo = (UserInputService.MouseBehavior == Enum.MouseBehavior.LockCenter)
                if shiftLockActivo then
                    humanoid.AutoRotate = false
                    local camLook = camera.CFrame.LookVector
                    local targetY = math.atan2(-camLook.X, -camLook.Z)
                    hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, targetY, 0)
                else
                    humanoid.AutoRotate = true
                end
                if humanoid.MoveDirection.Magnitude > 0.1 then
                    local speedMultiplier = getgenv().TPWalkSpeed * 55
                    hrp.AssemblyLinearVelocity = Vector3.new(
                        humanoid.MoveDirection.X * speedMultiplier,
                        hrp.AssemblyLinearVelocity.Y,
                        humanoid.MoveDirection.Z * speedMultiplier
                    )
                end
            end)
        end
    else
        if TPWalkConn then TPWalkConn:Disconnect(); TPWalkConn = nil end
        pcall(function()
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.Sit = false
                    humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, true)
                    humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
                    humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
                    humanoid:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, true)
                    humanoid.AutoRotate = true
                end
            end
        end)
    end
end

-- ⚔️ ATAQUE RÁPIDO
local function SetFlashAttack(s)
    FlashAttackEnabled = s
    if s then
        if not FlashConn then
            FlashConn = RunService.Heartbeat:Connect(function()
                if not FlashAttackEnabled or not Character then return end
                local tool = Character:FindFirstChildWhichIsA("Tool")
                if not tool then return end
                pcall(function()
                    for _ = 1, FlashMultiplier do tool:Activate() end
                    for _, v in pairs(tool:GetDescendants()) do
                        if v:IsA("NumberValue") or v:IsA("IntValue") then
                            local name = v.Name:lower()
                            if name:find("cooldown") or name:find("delay") or name:find("rate") or name:find("time") then
                                v.Value = 0
                            end
                        end
                    end
                end)
            end)
        end
    else
        if FlashConn then FlashConn:Disconnect(); FlashConn = nil end
    end
end

-- 🛡️ SIT PROTECTOR
local function GuardarEstadosOriginalesSit(Personaje)
    if not Personaje then return end
    local Hum = Personaje:FindFirstChildWhichIsA("Humanoid")
    if not Hum then return end
    SitEstadosOriginales = {
        Sit = false, PlatformStand = false, AutoRotate = true,
        Seated = Hum:GetStateEnabled(Enum.HumanoidStateType.Seated),
        FallingDown = Hum:GetStateEnabled(Enum.HumanoidStateType.FallingDown),
        Ragdoll = Hum:GetStateEnabled(Enum.HumanoidStateType.Ragdoll),
        PlatformStanding = Hum:GetStateEnabled(Enum.HumanoidStateType.PlatformStanding),
    }
end
local function RestaurarTodoNormalSit()
    local Pj = LocalPlayer.Character
    if not Pj then return end
    local Hum = Pj:FindFirstChildWhichIsA("Humanoid")
    if not Hum then return end
    Hum.Sit = SitEstadosOriginales.Sit
    Hum.PlatformStand = SitEstadosOriginales.PlatformStand
    Hum.AutoRotate = SitEstadosOriginales.AutoRotate
    Hum:SetStateEnabled(Enum.HumanoidStateType.Seated, SitEstadosOriginales.Seated)
    Hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, SitEstadosOriginales.FallingDown)
    Hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, SitEstadosOriginales.Ragdoll)
    Hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, SitEstadosOriginales.PlatformStanding)
end
local function CrearSitGui()
    if SitGui then return end
    local Gui = Instance.new("ScreenGui")
    Gui.Name = "SitProtector"; Gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling; Gui.ResetOnSpawn = false
    local Marco = Instance.new("Frame")
    Marco.Size = UDim2.new(0, 120, 0, 55); Marco.Position = UDim2.new(0.02, 0, 0.02, 0)
    Marco.BackgroundColor3 = Color3.fromRGB(255, 245, 190)
    Marco.BorderColor3 = Color3.fromRGB(240, 215, 110); Marco.BorderSizePixel = 2
    Marco.Active = true; Marco.Draggable = true; Marco.Parent = Gui
    local Boton = Instance.new("TextButton")
    Boton.Size = UDim2.new(0, 90, 0, 35); Boton.Position = UDim2.new(0.5, -45, 0.5, -17)
    Boton.BackgroundColor3 = Color3.fromRGB(250, 225, 130)
    Boton.TextColor3 = Color3.fromRGB(90, 70, 20); Boton.Font = Enum.Font.GothamBold
    Boton.TextSize = 12; Boton.Text = "OFF"; Boton.AutoLocalize = false; Boton.Parent = Marco
    SitGui = {Gui=Gui, Marco=Marco, Boton=Boton}
end
local function SetSitProtector(s)
    if s and SitProtectorEnabled then return end
    SitProtectorEnabled = s
    CrearSitGui()
    if s then
        SitGui.Boton.Text = "ON"
        SitGui.Boton.BackgroundColor3 = Color3.fromRGB(200, 245, 170)
        GuardarEstadosOriginalesSit(LocalPlayer.Character)
        SitConexiones.Bucle = RunService.Heartbeat:Connect(function()
            local Pj = LocalPlayer.Character
            if not Pj then return end
            local Hum = Pj:FindFirstChildWhichIsA("Humanoid")
            local Raiz = Pj:FindFirstChild("HumanoidRootPart")
            if not Hum or not Raiz then return end
            Hum.Sit = true
            Hum:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
            Hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            Hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            Hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, false)
            Hum.AutoRotate = true; Hum.PlatformStand = false
            if math.abs(Raiz.RotVelocity.Y) > 8 then
                Raiz.RotVelocity = Vector3.new(0, math.sign(Raiz.RotVelocity.Y) * 8, 0)
            end
            for _, Pieza in Pj:GetChildren() do
                if Pieza:IsA("Weld") or Pieza:IsA("WeldConstraint") or Pieza:IsA("Motor6D") then
                    if Pieza.Name ~= "RootJoint" and Pieza.Name ~= "Neck" and Pieza.Name ~= "Waist"
                    and not Pieza:FindFirstAncestorWhichIsA("Tool") then
                        Pieza:Destroy()
                    end
                end
            end
        end)
    else
        if SitGui then
            SitGui.Boton.Text = "OFF"
            SitGui.Boton.BackgroundColor3 = Color3.fromRGB(250, 225, 130)
        end
        if SitConexiones.Bucle then SitConexiones.Bucle:Disconnect(); SitConexiones.Bucle = nil end
        RestaurarTodoNormalSit()
    end
end
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.2)
    if SitGui and SitProtectorEnabled then SitGui.Gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end
end)

-- ESP / FULLBRIGHT / FPS BOOST
local function SetESP(state)
    ESPEnabled=state
    if state then
        if not ESPConn then
            ESPConn=RunService.Heartbeat:Connect(function()
                if not ESPEnabled then return end
                for _,plr in ipairs(Players:GetPlayers()) do
                    if plr~=LocalPlayer and plr.Character then
                        local char=plr.Character
                        if not char:FindFirstChild("WindESP_HL") then
                            local hl=Instance.new("Highlight"); hl.Name="WindESP_HL"; hl.FillColor=Color3.fromRGB(255,50,50); hl.OutlineColor=Color3.fromRGB(255,255,255); hl.FillTransparency=0.6; hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop; hl.Parent=char
                        end
                        local head=char:FindFirstChild("Head")
                        if head then
                            local bg=head:FindFirstChild("WindESP_BG")
                            if not bg then
                                bg=Instance.new("BillboardGui"); bg.Name="WindESP_BG"; bg.Size=UDim2.new(0,220,0,40); bg.StudsOffset=Vector3.new(0,2.5,0); bg.AlwaysOnTop=true
                                local tl=Instance.new("TextLabel"); tl.Name="TextLabel"; tl.Size=UDim2.new(1,0,1,0); tl.BackgroundTransparency=1; tl.TextColor3=Color3.fromRGB(255,255,255); tl.TextStrokeTransparency=0; tl.Font=Enum.Font.SourceSansBold; tl.TextScaled=true; tl.Parent=bg; bg.Parent=head
                            end
                            local dist = RootPart and math.floor((RootPart.Position-head.Position).Magnitude) or 0
                            bg.TextLabel.Text = plr.Name.."  ["..dist.." m]"
                        end
                    end
                end
            end)
        end
    else
        if ESPConn then ESPConn:Disconnect(); ESPConn=nil end
        for _,plr in ipairs(Players:GetPlayers()) do
            if plr.Character then
                local hl=plr.Character:FindFirstChild("WindESP_HL"); if hl then hl:Destroy() end
                local head=plr.Character:FindFirstChild("Head")
                if head then local bg=head:FindFirstChild("WindESP_BG"); if bg then bg:Destroy() end end
            end
        end
    end
end
local function SetFullbright(state)
    FullbrightEnabled=state
    if state then
        Lighting.ClockTime=14; Lighting.Brightness=2; Lighting.Ambient=Color3.fromRGB(255,255,255); Lighting.OutdoorAmbient=Color3.fromRGB(255,255,255); Lighting.FogEnd=100000; Lighting.GlobalShadows=false; Lighting.ExposureCompensation=0.5
        for _,e in ipairs(Lighting:GetChildren()) do pcall(function() e.Enabled=false end) end
    else
        Lighting.ClockTime=OriginalLight.ClockTime; Lighting.Brightness=OriginalLight.Brightness; Lighting.Ambient=OriginalLight.Ambient; Lighting.OutdoorAmbient=OriginalLight.OutdoorAmbient; Lighting.FogEnd=OriginalLight.FogEnd; Lighting.GlobalShadows=OriginalLight.GlobalShadows; Lighting.ExposureCompensation=OriginalLight.ExposureCompensation
        for _,e in ipairs(Lighting:GetChildren()) do pcall(function() e.Enabled=true end) end
    end
end
local function SetFpsBoost(state)
    FpsBoostEnabled=state
    if state then
        pcall(function() GameSettings.SavedQualityLevel=Enum.SavedQualitySetting.QualityLevel1 end)
        pcall(function() local t=workspace:FindFirstChildOfClass("Terrain"); if t then t.WaterWaveSize=0; t.WaterWaveSpeed=0 end end)
    else pcall(function() GameSettings.SavedQualityLevel=OriginalQuality end) end
end
local function ServerHop()
    task.spawn(function()
        local ok=pcall(function()
            local Http=game:GetService("HttpService")
            local response=Http:JSONDecode(Http:GetAsync("https://games.roproxy.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100"))
            local candidates={}
            if response and response.data then for _,s in ipairs(response.data) do if s.id~=game.JobId and s.playing and s.maxPlayers and s.playing<s.maxPlayers then table.insert(candidates,s) end end end
            if #candidates>0 then
                local chosen=candidates[math.random(1,#candidates)]
                WindUI:Notify({Title="Server Hop", Content="Conectando...", Duration=3})
                TeleportService:TeleportToPlaceInstance(game.PlaceId, chosen.id)
            else WindUI:Notify({Title="Server Hop", Content="No hay servidores", Duration=3}) end
        end)
        if not ok then WindUI:Notify({Title="Server Hop", Content="Error (proxy caído?)", Duration=3}) end
    end)
end
local function SetClickTP(s)
    ClickTPEnabled=s
    if s then
        if not ClickTPConn then ClickTPConn=UserInputService.InputBegan:Connect(function(input,gp)
            if not ClickTPEnabled or gp then return end
            if input.UserInputType==Enum.UserInputType.MouseButton2 then
                local Mouse=LocalPlayer:GetMouse()
                if RootPart and Mouse and Mouse.Hit then RootPart.CFrame=CFrame.new(Mouse.Hit.Position+Vector3.new(0,3,0)) end
            end
        end) end
    else if ClickTPConn then ClickTPConn:Disconnect(); ClickTPConn=nil end end
end
local function SetSpinBot(s)
    SpinBotEnabled=s
    if s then
        if not SpinConn then SpinConn=RunService.RenderStepped:Connect(function()
            if not SpinBotEnabled or not RootPart then return end
            RootPart.CFrame=RootPart.CFrame*CFrame.Angles(0,math.rad(15),0)
        end) end
    else if SpinConn then SpinConn:Disconnect(); SpinConn=nil end end
end
local function SetFreeze(s) FreezePositionEnabled=s; if RootPart then pcall(function() RootPart.Anchored=s end) end end
local function SetAutoWalk(s)
    AutoWalkEnabled=s
    if s then
        if not AutoWalkConn then AutoWalkConn=RunService.RenderStepped:Connect(function()
            if not AutoWalkEnabled or not Humanoid then return end
            local cam=workspace.CurrentCamera
            if cam then local dir=cam.CFrame.LookVector*Vector3.new(1,0,1); if dir.Magnitude>0 then Humanoid:Move(dir.Unit,false) end end
        end) end
    else
        if AutoWalkConn then AutoWalkConn:Disconnect(); AutoWalkConn=nil end
        if Humanoid then Humanoid:Move(Vector3.new(0,0,0)) end
    end
end
local function RemoveParticles()
    local count=0
    pcall(function() for _,v in ipairs(workspace:GetDescendants()) do if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") or v:IsA("Beam") or v:IsA("Explosion") then v:Destroy(); count=count+1 end end end)
    WindUI:Notify({Title="Partículas", Content="Eliminados "..count.." efectos", Duration=2})
end
local function SetAutoClicker(s)
    AutoClickerEnabled = s
    if s then
        task.spawn(function()
            while AutoClickerEnabled do
                pcall(function()
                    local VIM = game:GetService("VirtualInputManager")
                    local ml = UserInputService:GetMouseLocation()
                    VIM:SendMouseButtonEvent(ml.X, ml.Y, 0, true, game, 0)
                    task.wait(0.02)
                    VIM:SendMouseButtonEvent(ml.X, ml.Y, 0, false, game, 0)
                end)
                task.wait(math.max(0.05, 1 / AutoClickerCPS))
            end
        end)
    end
end
local function SetNoFallDamage(s)
    NoFallDamageEnabled = s
    if s then
        if not NoFallConn and Humanoid then
            NoFallConn = Humanoid.StateChanged:Connect(function(_, new)
                if NoFallDamageEnabled and new == Enum.HumanoidStateType.Landed then
                    pcall(function() if Humanoid then Humanoid.Health = Humanoid.MaxHealth end end)
                end
            end)
        end
    else
        if NoFallConn then NoFallConn:Disconnect(); NoFallConn = nil end
    end
end
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.5)
    if NoFallDamageEnabled then
        if NoFallConn then NoFallConn:Disconnect(); NoFallConn = nil end
        SetNoFallDamage(true)
    end
end)
local function SetAntiAFK(s)
    AntiAFKEnabled = s
    if s then
        if not AntiAFKConn then
            AntiAFKConn = LocalPlayer.Idled:Connect(function()
                pcall(function()
                    local vu = game:GetService("VirtualUser")
                    vu:CaptureController(); vu:ClickButton2(Vector2.new())
                end)
            end)
        end
    else
        if AntiAFKConn then AntiAFKConn:Disconnect(); AntiAFKConn = nil end
    end
end
local function TPAllToMe()
    if not RootPart then return end
    local count = 0
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            pcall(function() p.Character.HumanoidRootPart.CFrame = RootPart.CFrame * CFrame.new(math.random(-6,6), 0, math.random(-6,6)) end)
            count = count + 1
        end
    end
    WindUI:Notify({Title="TP All", Content="Teleportados "..count.." jugadores a ti", Duration=3})
end

-- 👻 INVISIBLE ULTRA
local function CollectHideParts()
    HideParts = {}
    if not Character then return end
    for _, obj in pairs(Character:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Transparency == 0 then table.insert(HideParts, obj) end
    end
end
local function SetHidePos(s)
    HidePosEnabled = s
    if s then
        CollectHideParts()
        for _, part in pairs(HideParts) do pcall(function() part.Transparency = 0.5 end) end
        if not HidePosConn then
            HidePosConn = RunService.Heartbeat:Connect(function()
                if not HidePosEnabled or not RootPart or not Humanoid then return end
                local cf = RootPart.CFrame
                local camOffset = Humanoid.CameraOffset
                local hidden = cf * CFrame.new(0, -200000, 0)
                RootPart.CFrame = hidden
                Humanoid.CameraOffset = hidden:ToObjectSpace(CFrame.new(cf.Position)).Position
                RunService.RenderStepped:Wait()
                RootPart.CFrame = cf
                Humanoid.CameraOffset = camOffset
            end)
        end
    else
        if HidePosConn then HidePosConn:Disconnect(); HidePosConn = nil end
        for _, part in pairs(HideParts) do pcall(function() part.Transparency = 0 end) end
        pcall(function() if Humanoid then Humanoid.CameraOffset = Vector3.new(0,0,0) end end)
    end
end
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.G then
        SetHidePos(not HidePosEnabled)
        pcall(function() WindUI:Notify({Title="Invisible Ultra", Content=HidePosEnabled and "Activado (tecla G)" or "Desactivado", Duration=2}) end)
    end
end)
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.5)
    if HidePosEnabled then
        CollectHideParts()
        for _, p in pairs(HideParts) do pcall(function() p.Transparency = 0.5 end) end
    end
end)

-- 🎈 BOTONES FLOTANTES
local BotonesFlotantes = {}
local FlotCount = 0
local function CrearFlotante(id, texto, colorOn, getEstado, alternar)
    FlotCount = FlotCount + 1
    local gui = Instance.new("ScreenGui")
    gui.Name = "Flotante_"..id; gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling; gui.Enabled = false
    local puesto = pcall(function()
        if gethui then gui.Parent = gethui() else gui.Parent = game:GetService("CoreGui") end
    end)
    if not puesto then pcall(function() gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end) end
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 54, 0, 54); btn.Position = UDim2.new(0.5, -100 + ((FlotCount-1)*64), 0.88, 0)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 50); btn.Text = texto
    btn.Font = Enum.Font.GothamBold; btn.TextColor3 = Color3.fromRGB(90, 70, 15)
    btn.TextSize = 13; btn.BorderSizePixel = 0; btn.Active = true
    btn.Draggable = true; btn.AutoLocalize = false; btn.Parent = gui
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1, 0)
    local stroke = Instance.new("UIStroke"); stroke.Thickness = 2.5; stroke.Parent = btn
    local function Actualizar()
        local on = getEstado()
        btn.BackgroundColor3 = on and colorOn or Color3.fromRGB(255, 248, 215)
        btn.Text = on and (texto.." ✓") or texto
        btn.TextColor3 = Color3.fromRGB(90, 70, 15)
        stroke.Color = on and Color3.fromRGB(200, 165, 40) or colorOn
    end
    Actualizar()
    btn.MouseButton1Click:Connect(function() alternar(not getEstado()); task.wait(0.05); Actualizar() end)
    BotonesFlotantes[id] = {Gui=gui, Actualizar=Actualizar}
end
CrearFlotante("Invisible", "INV", Color3.fromRGB(255, 235, 140), function() return HidePosEnabled end, function(s) SetHidePos(s) end)
CrearFlotante("TPWalk", "TPW", Color3.fromRGB(255, 242, 165), function() return TPWalkEnabled end, function(s) SetTPWalk(s) end)
CrearFlotante("Ataque", "ATA", Color3.fromRGB(250, 220, 110), function() return FlashAttackEnabled end, function(s) SetFlashAttack(s) end)
task.spawn(function()
    while task.wait(0.3) do
        for _, f in pairs(BotonesFlotantes) do if f.Gui.Enabled then f.Actualizar() end end
    end
end)

-- 🛡️ ESCUDOS REALES
local function SetAntiKick(s)
    AntiKickEnabled = s
    if s and not AntiKickHooked then
        local hooked = false
        pcall(function()
            if getrawmetatable and newcclosure and setreadonly and getnamecallmethod then
                local mt = getrawmetatable(game)
                OldNameCall = mt.__namecall
                setreadonly(mt, false)
                mt.__namecall = newcclosure(function(self, ...)
                    local method = getnamecallmethod()
                    if AntiKickEnabled and method == "Kick" then return nil end
                    return OldNameCall(self, ...)
                end)
                setreadonly(mt, true)
                hooked = true
            end
        end)
        if not hooked then
            pcall(function()
                if hookfunction and LocalPlayer and LocalPlayer.Kick then
                    local replacement = (newcclosure and newcclosure(function() end)) or function() end
                    hookfunction(LocalPlayer.Kick, replacement)
                    hooked = true
                end
            end)
        end
        AntiKickHooked = hooked
        if not hooked then
            WindUI:Notify({Title="Anti-Kick", Content="Tu executor no soporta el hook. Solo sirve contra kicks locales.", Duration=5})
        else
            WindUI:Notify({Title="Anti-Kick", Content="Hook activado (bloquea kicks locales).", Duration=3})
        end
    end
end
local function SetAntiReset(s)
    AntiResetEnabled = s
    if s then pcall(function() StarterGui:SetCore("ResetButtonCallback", false) end)
    else pcall(function() StarterGui:SetCore("ResetButtonCallback", function() if Humanoid then Humanoid.Health = 0 end end) end) end
end
local function SetAntiSit(s)
    AntiSitEnabled = s
    if not s and Humanoid then pcall(function() Humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, true) end) end
end
local function SetAntiFling(s) AntiFlingEnabled = s end
local function SetAntiFreeze(s) AntiFreezeEnabled = s end
local function SetAntiReport(s)
    AntiReportEnabled = s
    pcall(function() StarterGui:SetCore("ReportAbusePageEnabled", not s) end)
end
local function AntiVCForceReconnect()
    pcall(function() if VoiceChatService and VoiceChatService.JoinVoice then VoiceChatService:JoinVoice() end end)
    pcall(function() local internal = game:GetService("VoiceChatInternal"); if internal and internal.JoinVoice then internal:JoinVoice() end end)
    pcall(function() if VoiceChatService and VoiceChatService.SetVoiceChatEnabled then VoiceChatService:SetVoiceChatEnabled(true) end end)
end
local function SetAntiVC(s)
    AntiVCEnabled = s
    if s then
        if not AntiVCConn then AntiVCConn = RunService.Heartbeat:Connect(function() if not AntiVCEnabled then return end; AntiVCForceReconnect() end) end
        if not AntiVCFastConn then AntiVCFastConn = RunService.RenderStepped:Connect(function() if not AntiVCEnabled then return end; AntiVCForceReconnect() end) end
        task.spawn(function() for i = 1, 8 do if not AntiVCEnabled then break end; task.wait(0.1); AntiVCForceReconnect() end end)
        WindUI:Notify({Title="Anti-VC", Content="Anti-VC Ultra activado", Duration=3})
    else
        if AntiVCConn then AntiVCConn:Disconnect(); AntiVCConn = nil end
        if AntiVCFastConn then AntiVCFastConn:Disconnect(); AntiVCFastConn = nil end
    end
end

-- 🎯 MODULO TARGET
local function TargetSafe(fn) local st, r = pcall(fn); return st and r or nil end
local function TargetGetPing()
    return TargetSafe(function() return (game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()) / 1000 end) or 0.1
end
local function TargetGetRoot(p)
    return TargetSafe(function() local c = p and p.Character; if c and c:FindFirstChild("HumanoidRootPart") then return c.HumanoidRootPart end end)
end
local function TargetCurrent()
    if not TargetedPlayerName then return nil end
    return Players:FindFirstChild(TargetedPlayerName)
end
local function TargetTeleportTo(target)
    TargetSafe(function()
        local r = TargetGetRoot(LocalPlayer); local t = TargetGetRoot(target)
        if r and t then r.Velocity = Vector3.new(0,0,0); r.CFrame = CFrame.new(t.Position) + Vector3.new(0,2,0) end
    end)
end
local function TargetPredictionTP(target)
    TargetSafe(function()
        local r = TargetGetRoot(LocalPlayer); local t = TargetGetRoot(target)
        if not r or not t then return end
        local pos, vel = t.Position, t.Velocity
        local ping = TargetGetPing()
        r.CFrame = CFrame.new(pos.X + vel.X * ping * 3.5, pos.Y + vel.Y * ping * 2, pos.Z + vel.Z * ping * 3.5)
    end)
end
local function TargetPush(target)
    TargetSafe(function()
        local r = TargetGetRoot(LocalPlayer); local t = TargetGetRoot(target)
        if r and t then t.Velocity = (t.Position - r.Position).Unit * 100 + Vector3.new(0,50,0) end
    end)
end
local function TargetPlayAnim(id, time, speed)
    TargetSafe(function()
        local c = LocalPlayer.Character; if not c then return end
        local h = c:FindFirstChildOfClass("Humanoid"); if not h then return end
        local a = Instance.new("Animation"); a.AnimationId = "rbxassetid://"..id
        local tr = h:LoadAnimation(a); tr:Play(); tr.TimePosition = time; tr:AdjustSpeed(speed)
    end)
end
local function TargetStopAnim()
    TargetSafe(function()
        local c = LocalPlayer.Character; if not c then return end
        local h = c:FindFirstChildOfClass("Humanoid"); if not h then return end
        for _, tr in ipairs(h:GetPlayingAnimationTracks()) do tr:Stop() end
    end)
end
local function TargetBreakVel()
    return TargetSafe(function()
        local b = Instance.new("BodyAngularVelocity")
        b.Name = "BreakVel"; b.MaxTorque = Vector3.new(5e4,5e4,5e4); b.P = 1250
        return b
    end)
end
local function TargetCleanBreakVel()
    TargetSafe(function() local r = TargetGetRoot(LocalPlayer); if r and r:FindFirstChild("BreakVel") then r.BreakVel:Destroy() end end)
end
local function TargetCleanup()
    TargetCleanBreakVel(); TargetStopAnim()
    TargetSafe(function() if Humanoid then workspace.CurrentCamera.CameraSubject = Humanoid end end)
end
local function TargetAnyActive()
    for _, v in pairs(TargetToggles) do if v then return true end end
    return false
end
local function TargetEnsureLoop()
    if TargetLoopConn then return end
    TargetLoopConn = RunService.Heartbeat:Connect(function()
        if not TargetAnyActive() then return end
        local t = TargetCurrent(); if not t then return end
        local r = TargetGetRoot(LocalPlayer); if not r then return end
        local tr = TargetGetRoot(t); local char = t.Character
        if TargetToggles.Fling then TargetPredictionTP(t) end
        if TargetToggles.View then local h = char and char:FindFirstChildOfClass("Humanoid"); if h then workspace.CurrentCamera.CameraSubject = h end end
        if TargetToggles.Focus then TargetTeleportTo(t); TargetPush(t) end
        if TargetToggles.Bang then
            TargetPlayAnim(5918726674, 0, 1)
            if not r:FindFirstChild("BreakVel") then local bv = TargetBreakVel(); if bv then bv.Parent = r end end
            if tr then r.CFrame = tr.CFrame * CFrame.new(0,0,1.1); r.Velocity = Vector3.new() end
        end
        if TargetToggles.HeadSit then
            local head = char and char:FindFirstChild("Head")
            if head then
                if not r:FindFirstChild("BreakVel") then local bv = TargetBreakVel(); if bv then bv.Parent = r end end
                if Humanoid then Humanoid.Sit = true end
                r.CFrame = head.CFrame * CFrame.new(0,2,0); r.Velocity = Vector3.new()
            end
        end
        if TargetToggles.Stand then
            TargetPlayAnim(13823324057, 4, 0)
            if not r:FindFirstChild("BreakVel") then local bv = TargetBreakVel(); if bv then bv.Parent = r end end
            if tr then r.CFrame = tr.CFrame * CFrame.new(-3,1,0); r.Velocity = Vector3.new() end
        end
        if TargetToggles.Backpack then
            if not r:FindFirstChild("BreakVel") then local bv = TargetBreakVel(); if bv then bv.Parent = r end end
            if Humanoid then Humanoid.Sit = true end
            if tr then r.CFrame = tr.CFrame * CFrame.new(0,0,1.2) * CFrame.Angles(0,-3,0); r.Velocity = Vector3.new() end
        end
        if TargetToggles.Doggy then
            TargetPlayAnim(13694096724, 3.4, 0)
            local torso = char and (char:FindFirstChild("LowerTorso") or char:FindFirstChild("Torso"))
            if torso then
                if not r:FindFirstChild("BreakVel") then local bv = TargetBreakVel(); if bv then bv.Parent = r end end
                r.CFrame = torso.CFrame * CFrame.new(0,0.23,0); r.Velocity = Vector3.new()
            end
        end
        if TargetToggles.Drag then
            TargetPlayAnim(10714360343, 0.5, 0)
            local hand = char and (char:FindFirstChild("RightHand") or char:FindFirstChild("Right Arm"))
            if hand then
                if not r:FindFirstChild("BreakVel") then local bv = TargetBreakVel(); if bv then bv.Parent = r end end
                r.CFrame = hand.CFrame * CFrame.new(0,-2.5,1) * CFrame.Angles(-2,-3,0)
                r.Velocity = Vector3.new()
            end
        end
    end)
end
local function TargetToggle(key, s)
    TargetToggles[key] = s
    if s then
        if not TargetCurrent() then WindUI:Notify({Title="Target", Content="Selecciona un objetivo primero", Duration=3}) end
        TargetEnsureLoop()
    else
        if not TargetAnyActive() then TargetCleanup() end
    end
end
local function TargetUpdateInfo(p)
    if TargetInfoParagraph and TargetInfoParagraph.SetDesc then
        if p then TargetInfoParagraph:SetDesc(string.format("UserID: %d\nDisplay: %s\nAccountAge: %d días", p.UserId, p.DisplayName, p.AccountAge))
        else TargetInfoParagraph:SetDesc("UserID: —\nDisplay: —\nAccountAge: —") end
    end
    if p and TargetAvatarImg then
        pcall(function()
            local thumb = Players:GetUserThumbnailAsync(p.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)
            if TargetAvatarImg.SetImage then TargetAvatarImg:SetImage(thumb) end
        end)
    end
end
local function TargetSelect(name)
    if not name or name == "" or name == "No hay jugadores" or name == "Nadie (detener)" then
        TargetedPlayerName = nil
        for k in pairs(TargetToggles) do TargetToggles[k] = false end
        TargetCleanup(); TargetUpdateInfo(nil)
        WindUI:Notify({Title="Target", Content="Objetivo limpiado", Duration=2})
        return
    end
    local p = Players:FindFirstChild(name)
    if not p then
        local low = name:lower()
        for _, v in ipairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and (v.Name:lower():match(low) or v.DisplayName:lower():match(low)) then p = v; break end
        end
    end
    if p then
        if table.find(TargetWhitelist, p.UserId) then
            WindUI:Notify({Title="Target", Content=p.Name.." está en lista blanca (no se puede objetivo)", Duration=3})
            return
        end
        TargetedPlayerName = p.Name; TargetUpdateInfo(p)
        WindUI:Notify({Title="Target", Content="Objetivo: "..p.Name, Duration=2})
    else
        WindUI:Notify({Title="Target", Content="Jugador no encontrado", Duration=2})
    end
end
Players.PlayerRemoving:Connect(function(p)
    if p.Name == TargetedPlayerName then
        TargetedPlayerName = nil
        for k in pairs(TargetToggles) do TargetToggles[k] = false end
        TargetCleanup(); TargetUpdateInfo(nil)
        WindUI:Notify({Title="Target", Content="El objetivo salió del servidor", Duration=3})
    end
end)
_G.TargetModule = { SetTarget = function(n) TargetSelect(n) end, GetTarget = function() return TargetCurrent() end, Whitelist = TargetWhitelist }
getgenv().TargetModule = _G.TargetModule

-- 💾 AUTO-GUARDADO
local Saved = {}
local ConfigFolder = "DENJI_ALEX"
local ConfigFile = ConfigFolder .. "/Takemichi_Config.json"
local HttpService = game:GetService("HttpService")
local function Get(key, def) if Saved[key] ~= nil then return Saved[key] end; return def end

local ColorAccent = Color3.fromRGB(255, 235, 140)
local RefBordeCirculo, RefBordePerfil, RefTPBtn = nil, nil, nil
local function AplicarColor(c)
    ColorAccent = c
    pcall(function() if RefBordeCirculo then RefBordeCirculo.Color = c end end)
    pcall(function() if RefBordePerfil then RefBordePerfil.Color = c end end)
    pcall(function() if RefTPBtn then RefTPBtn.BackgroundColor3 = c end end)
    pcall(function() if Window and Window.SetAccent then Window:SetAccent(c) end end)
    pcall(function() if Window and Window.SetThemeColor then Window:SetThemeColor(c) end end)
    pcall(SaveConfig)
end

local function BuildConfig()
    return {
        ThemeVersion = 2,
        FlashAttackEnabled = FlashAttackEnabled, FlashMultiplier = FlashMultiplier,
        TPWalkEnabled = TPWalkEnabled, TPWalkSpeed = TPWalkSpeed,
        NoFrictionEnabled = NoFrictionEnabled, InvisibleEnabled = InvisibleEnabled,
        HidePosEnabled = HidePosEnabled, FlyEnabled = FlyEnabled, FlySpeed = FlySpeed,
        NoclipEnabled = NoclipEnabled, InfJumpEnabled = InfJumpEnabled,
        WalkSpeed = Humanoid and Humanoid.WalkSpeed or 16,
        JumpPower = Humanoid and Humanoid.JumpPower or 50,
        GravityScale = Humanoid and Humanoid.GravityScale or 1,
        AutoWalkEnabled = AutoWalkEnabled, FullbrightEnabled = FullbrightEnabled,
        ESPEnabled = ESPEnabled, FpsBoostEnabled = FpsBoostEnabled,
        FOV = (function() local ok,v=pcall(function() return workspace.CurrentCamera.FieldOfView end); return ok and v or 70 end)(),
        ClockTime = Lighting.ClockTime,
        AnimationSpeed = Humanoid and Humanoid.AnimationSpeed or 1,
        BodyScale = Humanoid and Humanoid.BodyHeightScale or 1,
        AutoRejoinEnabled = AutoRejoinEnabled, AntiVoidEnabled = AntiVoidEnabled,
        AntiRagdollEnabled = AntiRagdollEnabled, GodModeEnabled = GodModeEnabled,
        InstantRespawnEnabled = InstantRespawnEnabled, FollowPlayerEnabled = FollowPlayerEnabled,
        ClickTPEnabled = ClickTPEnabled, AutoJumpEnabled = AutoJumpEnabled,
        WalkOnWaterEnabled = WalkOnWaterEnabled, SpinBotEnabled = SpinBotEnabled,
        FreezePositionEnabled = FreezePositionEnabled, FallSpeedCap = FallSpeedCap,
        AntiAFKEnabled = AntiAFKEnabled, NoPushEnabled = NoPushEnabled,
        NoKnockbackEnabled = NoKnockbackEnabled, SitProtectorEnabled = SitProtectorEnabled,
        AutoClickerEnabled = AutoClickerEnabled, AutoClickerCPS = AutoClickerCPS,
        NoFallDamageEnabled = NoFallDamageEnabled, AntiKickEnabled = AntiKickEnabled,
        AntiResetEnabled = AntiResetEnabled, AntiSitEnabled = AntiSitEnabled,
        AntiFlingEnabled = AntiFlingEnabled, AntiFreezeEnabled = AntiFreezeEnabled,
        AntiReportEnabled = AntiReportEnabled, AntiVCEnabled = AntiVCEnabled,
        ServidoresVisitados = ServidoresVisitados,
        AccentR = math.floor(ColorAccent.R*255), AccentG = math.floor(ColorAccent.G*255), AccentB = math.floor(ColorAccent.B*255),
    }
end
local function DoSave()
    pcall(function()
        if not isfolder(ConfigFolder) then makefolder(ConfigFolder) end
        writefile(ConfigFile, HttpService:JSONEncode(BuildConfig()))
    end)
end
local SaveScheduled = false
local function SaveConfig()
    if SaveScheduled then return end
    SaveScheduled = true
    task.delay(0.4, function() SaveScheduled = false; DoSave() end)
end
local function AS(fn)
    return function(...)
        if fn then fn(...) end
        pcall(SaveConfig)
    end
end

local function LoadConfig()
    pcall(function()
        if isfolder(ConfigFolder) and isfile(ConfigFile) then
            local data = HttpService:JSONDecode(readfile(ConfigFile))
            if data then Saved = data end
        end
    end)
    if Saved.ThemeVersion ~= 2 then Saved.AccentR, Saved.AccentG, Saved.AccentB = nil, nil, nil end
    if Saved.ServidoresVisitados and type(Saved.ServidoresVisitados) == "table" then ServidoresVisitados = Saved.ServidoresVisitados end
    if Saved.FlashMultiplier then FlashMultiplier = Saved.FlashMultiplier end
    if Saved.TPWalkSpeed then TPWalkSpeed = Saved.TPWalkSpeed end; getgenv().TPWalkSpeed = TPWalkSpeed
    if Saved.FlySpeed then FlySpeed = Saved.FlySpeed end
    if Saved.FallSpeedCap then FallSpeedCap = Saved.FallSpeedCap end
    if Saved.AutoClickerCPS then AutoClickerCPS = Saved.AutoClickerCPS end
    if Saved.AccentR then ColorAccent = Color3.fromRGB(Saved.AccentR, Saved.AccentG or 235, Saved.AccentB or 140) end
end

local function AplicarConfiguracion()
    if not next(Saved) then return end
    local function Try(fn) pcall(fn) end
    Try(function() if Humanoid and Saved.WalkSpeed then Humanoid.WalkSpeed = Saved.WalkSpeed end end)
    Try(function() if Humanoid and Saved.JumpPower then Humanoid.JumpPower = Saved.JumpPower end end)
    Try(function() if Humanoid and Saved.GravityScale then Humanoid.GravityScale = Saved.GravityScale end end)
    Try(function() if Humanoid and Saved.AnimationSpeed then pcall(function() Humanoid.AnimationSpeed = Saved.AnimationSpeed end) end end)
    Try(function() if Humanoid and Saved.BodyScale then pcall(function() Humanoid.BodyHeightScale=Saved.BodyScale; Humanoid.BodyWidthScale=Saved.BodyScale; Humanoid.BodyDepthScale=Saved.BodyScale; if Humanoid.HeadScale then Humanoid.HeadScale=Saved.BodyScale end end) end end)
    Try(function() if Saved.FOV then pcall(function() workspace.CurrentCamera.FieldOfView = Saved.FOV end) end end)
    Try(function() if Saved.ClockTime then Lighting.ClockTime = Saved.ClockTime end end)
    Try(function() if Saved.InvisibleEnabled and Character then for _,v in pairs(Character:GetDescendants()) do if v:IsA("BasePart") then v.LocalTransparencyModifier=1 end end; InvisibleEnabled = true end end)
    Try(function() if Saved.HidePosEnabled then SetHidePos(true) end end)
    Try(function() if Saved.FlashAttackEnabled then SetFlashAttack(true) end end)
    Try(function() if Saved.TPWalkEnabled then SetTPWalk(true) end end)
    Try(function() if Saved.FlyEnabled then StartFly() end end)
    Try(function() if Saved.NoclipEnabled then SetNoclip(true) end end)
    Try(function() if Saved.InfJumpEnabled then SetInfJump(true) end end)
    Try(function() if Saved.AutoWalkEnabled then SetAutoWalk(true) end end)
    Try(function() if Saved.FullbrightEnabled then SetFullbright(true) end end)
    Try(function() if Saved.ESPEnabled then SetESP(true) end end)
    Try(function() if Saved.FpsBoostEnabled then SetFpsBoost(true) end end)
    Try(function() if Saved.ClickTPEnabled then SetClickTP(true) end end)
    Try(function() if Saved.SpinBotEnabled then SetSpinBot(true) end end)
    Try(function() if Saved.SitProtectorEnabled then SetSitProtector(true) end end)
    Try(function() if Saved.AutoClickerEnabled then SetAutoClicker(true) end end)
    Try(function() if Saved.NoFallDamageEnabled then SetNoFallDamage(true) end end)
    Try(function() if Saved.AntiAFKEnabled then SetAntiAFK(true) end end)
    Try(function() if Saved.FreezePositionEnabled and RootPart then pcall(function() RootPart.Anchored=true end); FreezePositionEnabled=true end end)
    Try(function() if Saved.NoFrictionEnabled~=nil then NoFrictionEnabled=Saved.NoFrictionEnabled end end)
    Try(function() if Saved.NoPushEnabled~=nil then NoPushEnabled=Saved.NoPushEnabled end end)
    Try(function() if Saved.NoKnockbackEnabled~=nil then NoKnockbackEnabled=Saved.NoKnockbackEnabled end end)
    Try(function() if Saved.AntiVoidEnabled~=nil then AntiVoidEnabled=Saved.AntiVoidEnabled end end)
    Try(function() if Saved.AntiRagdollEnabled~=nil then AntiRagdollEnabled=Saved.AntiRagdollEnabled end end)
    Try(function() if Saved.GodModeEnabled~=nil then GodModeEnabled=Saved.GodModeEnabled end end)
    Try(function() if Saved.AutoRejoinEnabled~=nil then AutoRejoinEnabled=Saved.AutoRejoinEnabled end end)
    Try(function() if Saved.InstantRespawnEnabled~=nil then InstantRespawnEnabled=Saved.InstantRespawnEnabled end end)
    Try(function() if Saved.FollowPlayerEnabled~=nil then FollowPlayerEnabled=Saved.FollowPlayerEnabled end end)
    Try(function() if Saved.AutoJumpEnabled~=nil then AutoJumpEnabled=Saved.AutoJumpEnabled end end)
    Try(function() if Saved.WalkOnWaterEnabled~=nil then WalkOnWaterEnabled=Saved.WalkOnWaterEnabled end end)
    Try(function() if Saved.AntiKickEnabled then SetAntiKick(true) end end)
    Try(function() if Saved.AntiResetEnabled then SetAntiReset(true) end end)
    Try(function() if Saved.AntiReportEnabled then SetAntiReport(true) end end)
    Try(function() if Saved.AntiVCEnabled then SetAntiVC(true) end end)
    Try(function() if Saved.AntiSitEnabled~=nil then AntiSitEnabled=Saved.AntiSitEnabled end end)
    Try(function() if Saved.AntiFlingEnabled~=nil then AntiFlingEnabled=Saved.AntiFlingEnabled end end)
    Try(function() if Saved.AntiFreezeEnabled~=nil then AntiFreezeEnabled=Saved.AntiFreezeEnabled end end)
end

LoadConfig()

-- VENTANA PRINCIPAL
local Window = WindUI:CreateWindow({
    Title="TKSH🍰", Icon="sword", Author="CHOCOFRZ🍓", Folder="DENJI•ALEX",
    Size=UDim2.fromOffset(600,540), MinSize=Vector2.new(520,420), MaxSize=Vector2.new(850,680),
    Transparent=true, Theme="Dark", Resizable=true, SideBarWidth=160,
    Background = "rbxassetid://81943489858207",
    BackgroundImageTransparency = 0.65,
    BackgroundColor3 = Color3.fromRGB(255, 253, 240),
    HideSearchBar = true, Accent=Color3.fromRGB(255, 235, 140),
    OpenButton={Title="CHOCOFRZ 🍓", Icon="sword", Enabled=true, Draggable=true, OnlyMobile=false, CornerRadius=UDim.new(1,0), StrokeThickness=2, StrokeColor=Color3.fromRGB(255, 235, 140), Scale=1},
})
pcall(function() AplicarColor(ColorAccent) end)

-- 1. PLAYER
local PlayerTab = Window:Tab({Title="Player", Icon="user"})
PlayerTab:Space({Size=6})
local StatsGroup = PlayerTab:Group({})
StatsGroup:Space({Size=10})
local PerfSection = StatsGroup:Section({Title="Rendimiento", Box=true, BoxBorder=true, Opened=true})
local StatsParagraph = PerfSection:Paragraph({Title="En Tiempo Real", Desc="Ping: Cargando...\nFPS: Cargando...\nMemoria: Cargando...", Image="activity", ImageSize=14})
task.spawn(function()
    while task.wait(1) do
        local ps, Ping = pcall(function() return math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()) end)
        local FPS = math.floor(1/RunService.Heartbeat:Wait())
        local Memory = math.floor(game:GetService("Stats"):GetTotalMemoryUsageMb())
        if StatsParagraph and StatsParagraph.SetDesc then StatsParagraph:SetDesc("Ping: "..(ps and (Ping.." ms") or "N/A").."\nFPS: "..FPS.."\nMemoria: "..Memory.." MB") end
    end
end)

-- 2. MAIN
local MainTab = Window:Tab({Title="Main", Icon="home"})
MainTab:Section({Title="Funciones Principales", TextSize=20}); MainTab:Space({Size=6})
MainTab:Section({Title="⚔️ Ataque Rápido", TextSize=18}); MainTab:Space({Size=6})
local FlashRow = MainTab:Group({})
FlashRow:Toggle({Title="Activar Ataque Rápido", Def=Get("FlashAttackEnabled", false), Callback=AS(SetFlashAttack)})
FlashRow:Space({Size=8})
FlashRow:Slider({Title="Multiplicador", Step=1, Value={Min=1,Max=30,Default=Get("FlashMultiplier", 5)}, Callback=AS(function(v) FlashMultiplier = v end)})
MainTab:Space({Size=12})
local MainRow1 = MainTab:Group({})
MainRow1:Button({Title="Teleport a Ti", Icon="map-pin", Justify="Center", Callback=function() local m=LocalPlayer:GetMouse(); if RootPart then RootPart.CFrame=CFrame.new(m.Hit.Position+Vector3.new(0,3,0)) end end})
MainRow1:Space({Size=8})
MainRow1:Toggle({Title="TPWalk (Bypass)", Def=Get("TPWalkEnabled", false), Callback=AS(SetTPWalk)})
MainTab:Space({Size=8})
MainTab:Slider({Title="Velocidad TPWalk", Step=0.05, Value={Min=0.01, Max=20, Default=Get("TPWalkSpeed", 0.30)}, Callback=AS(function(v) TPWalkSpeed = v; getgenv().TPWalkSpeed = v end)})
MainTab:Space({Size=8})
local MainRow2 = MainTab:Group({})
MainRow2:Toggle({Title="Salto Alto", Def=false, Callback=AS(function(s) if Humanoid then Humanoid.JumpPower=s and 120 or 50 end end)})
MainRow2:Space({Size=8})
MainRow2:Toggle({Title="Salto Infinito", Def=Get("InfJumpEnabled", false), Callback=AS(function(s) SetInfJump(s) end)})
MainTab:Space({Size=8})
local MainRow3 = MainTab:Group({})
MainRow3:Toggle({Title="Sin Fricción", Def=Get("NoFrictionEnabled", false), Callback=AS(function(s) NoFrictionEnabled=s end)})
MainRow3:Space({Size=8})
MainRow3:Toggle({Title="Sin Gravedad", Def=false, Callback=AS(function(s) if Humanoid then Humanoid.GravityScale=s and 0 or 1 end end)})
MainTab:Space({Size=8})
local MainRow4 = MainTab:Group({})
MainRow4:Toggle({Title="Invisible (Local)", Def=Get("InvisibleEnabled", false), Callback=AS(function(s) InvisibleEnabled=s; if Character then for _,v in pairs(Character:GetDescendants()) do if v:IsA("BasePart") then v.LocalTransparencyModifier=s and 1 or 0 end end end end)})
MainRow4:Space({Size=8})
MainRow4:Toggle({Title="Invisible Ultra (Oculta Posición) [G]", Def=Get("HidePosEnabled", false), Callback=AS(SetHidePos)})
MainTab:Space({Size=12})
MainTab:Section({Title="🎈 Botones Flotantes (tocá para prender/apagar)", TextSize=18}); MainTab:Space({Size=6})
local FlotSec = MainTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
FlotSec:Toggle({Title="Mostrar Botón: Invisible Ultra", Def=false, Callback=function(s) if BotonesFlotantes.Invisible then BotonesFlotantes.Invisible.Gui.Enabled = s end end})
FlotSec:Space({Size=6})
FlotSec:Toggle({Title="Mostrar Botón: TPWalk (Bypass)", Def=false, Callback=function(s) if BotonesFlotantes.TPWalk then BotonesFlotantes.TPWalk.Gui.Enabled = s end end})
FlotSec:Space({Size=6})
FlotSec:Toggle({Title="Mostrar Botón: Ataque Rápido", Def=false, Callback=function(s) if BotonesFlotantes.Ataque then BotonesFlotantes.Ataque.Gui.Enabled = s end end})
MainTab:Space({Size=12})
MainTab:Section({Title="Ajustes de Movimiento", TextSize=18}); MainTab:Space({Size=6})
local MainSliders = MainTab:Section({Title="Sliders Rápidos", Box=true, BoxBorder=true, Opened=true})
MainSliders:Slider({Title="WalkSpeed", Step=1, Value={Min=16,Max=250,Default=Get("WalkSpeed", 16)}, Callback=AS(function(v) if Humanoid then Humanoid.WalkSpeed=v end end)}); MainSliders:Space({Size=6})
MainSliders:Slider({Title="JumpPower", Step=1, Value={Min=50,Max=350,Default=Get("JumpPower", 50)}, Callback=AS(function(v) if Humanoid then Humanoid.JumpPower=v end end)}); MainSliders:Space({Size=6})
MainSliders:Slider({Title="Gravedad", Step=0.1, Value={Min=0,Max=2,Default=Get("GravityScale", 1)}, Callback=AS(function(v) if Humanoid then Humanoid.GravityScale=v end end)}); MainSliders:Space({Size=6})
MainSliders:Toggle({Title="Auto-Caminar (Hacia Adelante)", Def=Get("AutoWalkEnabled", false), Callback=AS(SetAutoWalk)})
MainTab:Space({Size=12})
MainTab:Section({Title="Acciones Rápidas", TextSize=18}); MainTab:Space({Size=6})
local MainA1 = MainTab:Group({})
MainA1:Button({Title="Server Hop", Icon="shuffle", Justify="Center", Callback=ServerHop}); MainA1:Space({Size=8})
MainA1:Button({Title="Rejoin (Mismo Server)", Icon="refresh-cw", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end})
MainTab:Space({Size=8})
local MainA2 = MainTab:Group({})
MainA2:Button({Title="Copiar Coordenadas", Icon="clipboard", Justify="Center", Callback=function() if RootPart then local p=RootPart.Position; setclipboard(math.floor(p.X)..", "..math.floor(p.Y)..", "..math.floor(p.Z)); WindUI:Notify({Title="Copiado", Content="Coordenadas copiadas", Duration=2}) end end})
MainA2:Space({Size=8})
MainA2:Button({Title="TP desde Portapapeles", Icon="map-pin", Justify="Center", Callback=function() local clip=""; pcall(function() clip=getclipboard() end); if not clip or clip=="" then WindUI:Notify({Title="Error", Content="Portapapeles vacío", Duration=2}) else TeleportToCoords(clip) end end})
MainTab:Space({Size=8})
local MainA3 = MainTab:Group({})
MainA3:Button({Title="Volver al Spawn", Icon="home", Justify="Center", Callback=function() if RootPart then RootPart.CFrame=SpawnCFrame; RootPart.Velocity=Vector3.new(0,0,0); WindUI:Notify({Title="Spawn", Content="Volviste al spawn", Duration=2}) end end})
MainA3:Space({Size=8})
MainA3:Button({Title="Reiniciar Personaje", Icon="refresh-cw", Justify="Center", Callback=function() if Character and Humanoid then Humanoid.Health=0 end end})
MainTab:Space({Size=12})
MainTab:Section({Title="📦 Scripts Externos", TextSize=18}); MainTab:Space({Size=6})
local ExtScripts = MainTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
ExtScripts:Button({Title="Hitbox Boys", Desc="Ejecutar script", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://pastebin.com/raw/4vL0qwVd"))(); WindUI:Notify({Title="Hitbox Boys", Content="Script ejecutado", Duration=3}) end)
end})
ExtScripts:Space({Size=10})
ExtScripts:Button({Title="Hitbox Girls", Desc="Ejecutar script", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://pastebin.com/raw/x9ivUUsh"))(); WindUI:Notify({Title="Hitbox Girls", Content="Script ejecutado", Duration=3}) end)
end})

-- 3. GAME
local GameTab = Window:Tab({Title="Game", Icon="gamepad-2"})
GameTab:Section({Title="Funciones Universales", TextSize=20}); GameTab:Space({Size=6})
GameTab:Section({Title="🚀 Movimiento", TextSize=18}); GameTab:Space({Size=6})
local GMov = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GMov:Toggle({Title="Fly (Volar)", Def=Get("FlyEnabled", false), Callback=AS(function(s) if s then StartFly() else StopFly() end end)}); GMov:Space({Size=6})
GMov:Toggle({Title="Noclip", Def=Get("NoclipEnabled", false), Callback=AS(SetNoclip)}); GMov:Space({Size=6})
GMov:Toggle({Title="Salto Infinito", Def=Get("InfJumpEnabled", false), Callback=AS(SetInfJump)}); GMov:Space({Size=6})
GMov:Toggle({Title="Click TP (Clic Der.)", Def=Get("ClickTPEnabled", false), Callback=AS(SetClickTP)}); GMov:Space({Size=6})
GMov:Toggle({Title="Auto-Jump (Bunny Hop)", Def=Get("AutoJumpEnabled", false), Callback=AS(function(s) AutoJumpEnabled=s end)}); GMov:Space({Size=6})
GMov:Toggle({Title="Walk on Water", Def=Get("WalkOnWaterEnabled", false), Callback=AS(function(s) WalkOnWaterEnabled=s end)}); GMov:Space({Size=6})
GMov:Toggle({Title="Spin Bot", Def=Get("SpinBotEnabled", false), Callback=AS(SetSpinBot)}); GMov:Space({Size=6})
GMov:Toggle({Title="Freeze Position", Def=Get("FreezePositionEnabled", false), Callback=AS(SetFreeze)}); GMov:Space({Size=6})
GMov:Slider({Title="WalkSpeed", Step=1, Value={Min=16,Max=250,Default=Get("WalkSpeed",16)}, Callback=AS(function(v) if Humanoid then Humanoid.WalkSpeed=v end end)}); GMov:Space({Size=6})
GMov:Slider({Title="JumpPower", Step=1, Value={Min=50,Max=350,Default=Get("JumpPower",50)}, Callback=AS(function(v) if Humanoid then Humanoid.JumpPower=v end end)}); GMov:Space({Size=6})
GMov:Slider({Title="Gravedad", Step=0.1, Value={Min=0,Max=2,Default=Get("GravityScale",1)}, Callback=AS(function(v) if Humanoid then Humanoid.GravityScale=v end end)})
GameTab:Space({Size=10})
GameTab:Section({Title="👁️ Visual", TextSize=18}); GameTab:Space({Size=6})
local GVis = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GVis:Toggle({Title="ESP Jugadores", Def=Get("ESPEnabled", false), Callback=AS(SetESP)}); GVis:Space({Size=6})
GVis:Toggle({Title="Fullbright", Def=Get("FullbrightEnabled", false), Callback=AS(SetFullbright)}); GVis:Space({Size=6})
GVis:Toggle({Title="FPS Boost", Def=Get("FpsBoostEnabled", false), Callback=AS(SetFpsBoost)}); GVis:Space({Size=6})
GVis:Slider({Title="FOV de Cámara", Step=1, Value={Min=60,Max=120,Default=Get("FOV",70)}, Callback=AS(function(v) pcall(function() workspace.CurrentCamera.FieldOfView=v end) end)}); GVis:Space({Size=6})
local GVisB = GVis:Group({})
GVisB:Button({Title="Desbloquear Zoom", Icon="zoom-in", Justify="Center", Callback=function() pcall(function() workspace.CurrentCamera.CameraMaxZoomDistance=1000; workspace.CurrentCamera.CameraMinZoomDistance=0.5 end); WindUI:Notify({Title="Zoom", Content="Desbloqueado", Duration=2}) end})
GVisB:Space({Size=8})
GVisB:Button({Title="Eliminar Partículas", Icon="trash-2", Justify="Center", Callback=RemoveParticles})
GameTab:Space({Size=10})
GameTab:Section({Title="🛡️ Protección", TextSize=18}); GameTab:Space({Size=6})
local GProt = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GProt:Toggle({Title="God Mode (Local)", Def=Get("GodModeEnabled", false), Callback=AS(function(s) GodModeEnabled=s end)}); GProt:Space({Size=6})
GProt:Toggle({Title="Anti-Void", Def=Get("AntiVoidEnabled", false), Callback=AS(function(s) AntiVoidEnabled=s end)}); GProt:Space({Size=6})
GProt:Toggle({Title="Anti-Ragdoll", Def=Get("AntiRagdollEnabled", false), Callback=AS(function(s) AntiRagdollEnabled=s end)}); GProt:Space({Size=6})
GProt:Toggle({Title="Anti-AFK (Real)", Def=Get("AntiAFKEnabled", false), Callback=AS(SetAntiAFK)}); GProt:Space({Size=6})
GProt:Toggle({Title="No Fall Damage", Def=Get("NoFallDamageEnabled", false), Callback=AS(SetNoFallDamage)}); GProt:Space({Size=6})
GProt:Toggle({Title="Auto-Respawn Instantáneo", Def=Get("InstantRespawnEnabled", false), Callback=AS(function(s) InstantRespawnEnabled=s end)}); GProt:Space({Size=6})
GProt:Toggle({Title="Auto-Rejoin al Morir", Def=Get("AutoRejoinEnabled", false), Callback=AS(function(s) AutoRejoinEnabled=s end)})
GameTab:Space({Size=10})
GameTab:Section({Title="🔧 Utilidades", TextSize=18}); GameTab:Space({Size=6})
local GUtil = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GUtil:Toggle({Title="Auto-Clicker", Def=Get("AutoClickerEnabled", false), Callback=AS(SetAutoClicker)}); GUtil:Space({Size=6})
GUtil:Slider({Title="CPS del Auto-Clicker", Step=1, Value={Min=1,Max=20,Default=Get("AutoClickerCPS",10)}, Callback=AS(function(v) AutoClickerCPS=v end)}); GUtil:Space({Size=8})
local GU1 = GUtil:Group({})
GU1:Button({Title="TP All a Mí", Icon="users", Justify="Center", Callback=TPAllToMe}); GU1:Space({Size=8})
GU1:Button({Title="Rejoin", Icon="refresh-cw", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end})
GUtil:Space({Size=6})
local GU2 = GUtil:Group({})
GU2:Button({Title="Server Hop", Icon="shuffle", Justify="Center", Callback=ServerHop}); GU2:Space({Size=8})
GU2:Button({Title="Copiar JobID", Icon="clipboard", Justify="Center", Callback=function() setclipboard(tostring(game.JobId)); WindUI:Notify({Title="Copiado", Content="JobID copiado", Duration=2}) end})
GUtil:Space({Size=6})
GUtil:Button({Title="Copiar Link del Juego", Icon="link", Justify="Center", Callback=function() setclipboard("https://www.roblox.com/games/"..tostring(game.PlaceId)); WindUI:Notify({Title="Copiado", Content="Link copiado", Duration=2}) end})
GameTab:Space({Size=10})
GameTab:Section({Title="📦 Scripts Universales", TextSize=18}); GameTab:Space({Size=6})
local GScr = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GScr:Button({Title="Infinite Yield (Admin Universal)", Desc="Ejecutar", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))(); WindUI:Notify({Title="Infinite Yield", Content="Script ejecutado", Duration=3}) end)
end})
GScr:Space({Size=10})
GScr:Button({Title="Dex Explorer", Desc="Ejecutar", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://raw.githubusercontent.com/infyiff/backup/main/dex.lua"))(); WindUI:Notify({Title="Dex Explorer", Content="Script ejecutado", Duration=3}) end)
end})
GameTab:Space({Size=12})

-- 4. TARGET
local TargetTab = Window:Tab({Title="Target", Icon="crosshair"})
TargetTab:Section({Title="🎯 Seleccionar Objetivo", TextSize=20}); TargetTab:Space({Size=6})
local TargetSel = TargetTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
TargetAvatarImg = TargetSel:Image({Image="rbxassetid://10818605405", ImageSize=80})
TargetSel:Space({Size=6})
TargetDropdown = TargetSel:Dropdown({Title="Jugador Objetivo", Values=GetPlayerNames(false), Value=1, Callback=function(s) TargetSelect(s) end})
TargetSel:Space({Size=6})
pcall(function() TargetSel:TextBox({Title="Nombre (o parte del display)", PlaceholderText="@nombre...", Callback=function(t) TargetNameInput = t end}) end)
TargetSel:Space({Size=6})
local TargetBtnRow = TargetSel:Group({})
TargetBtnRow:Button({Title="Seleccionar por Nombre", Icon="user-check", Justify="Center", Callback=function() TargetSelect(TargetNameInput or "") end})
TargetBtnRow:Space({Size=8})
TargetBtnRow:Button({Title="Herramienta de Clic", Icon="mouse-pointer", Justify="Center", Callback=function()
    pcall(function()
        local tool = Instance.new("Tool"); tool.Name = "TargetSelector"; tool.RequiresHandle = false; tool.TextureId = "rbxassetid://2716591855"
        tool.Activated:Connect(function()
            local m = LocalPlayer:GetMouse(); local hit = m and m.Target
            if not hit then return end
            local model = hit.Parent
            if model and model:IsA("Accessory") then model = model.Parent end
            if model and model:IsA("Model") then local p = Players:GetPlayerFromCharacter(model); if p then TargetSelect(p.Name) end end
        end)
        tool.Parent = LocalPlayer.Backpack
        WindUI:Notify({Title="Target", Content="Herramienta equipada: haz clic en un jugador", Duration=3})
    end)
end})
TargetSel:Space({Size=6})
TargetSel:Button({Title="Recargar Lista de Jugadores", Icon="refresh-cw", Justify="Center", Callback=function()
    pcall(function() if TargetDropdown then TargetDropdown:Refresh(GetPlayerNames(false)) end end)
    WindUI:Notify({Title="Target", Content="Lista recargada", Duration=2})
end})
TargetSel:Space({Size=6})
TargetInfoParagraph = TargetSel:Paragraph({Title="Información del Objetivo", Desc="UserID: —\nDisplay: —\nAccountAge: —", Image="info", ImageSize=14})
TargetTab:Space({Size=10})
TargetTab:Section({Title="🔄 Toggles de Objetivo (2 por línea)", TextSize=18}); TargetTab:Space({Size=6})
local TargetTog = TargetTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
local TT1 = TargetTog:Group({})
TT1:Toggle({Title="Lanzar (Fling)", Def=false, Callback=AS(function(s) TargetToggle("Fling", s) end)}); TT1:Space({Size=8})
TT1:Toggle({Title="Ver (Cámara)", Def=false, Callback=AS(function(s) TargetToggle("View", s) end)})
TargetTog:Space({Size=6})
local TT2 = TargetTog:Group({})
TT2:Toggle({Title="Enfocar (Focus)", Def=false, Callback=AS(function(s) TargetToggle("Focus", s) end)}); TT2:Space({Size=8})
TT2:Toggle({Title="Bang / Pegar", Def=false, Callback=AS(function(s) TargetToggle("Bang", s) end)})
TargetTog:Space({Size=6})
local TT3 = TargetTog:Group({})
TT3:Toggle({Title="Sentar en Cabeza", Def=false, Callback=AS(function(s) TargetToggle("HeadSit", s) end)}); TT3:Space({Size=8})
TT3:Toggle({Title="Pararse Junto (Stand)", Def=false, Callback=AS(function(s) TargetToggle("Stand", s) end)})
TargetTog:Space({Size=6})
local TT4 = TargetTog:Group({})
TT4:Toggle({Title="Mochila (Backpack)", Def=false, Callback=AS(function(s) TargetToggle("Backpack", s) end)}); TT4:Space({Size=8})
TT4:Toggle({Title="Posición Baja (Doggy)", Def=false, Callback=AS(function(s) TargetToggle("Doggy", s) end)})
TargetTog:Space({Size=6})
local TT5 = TargetTog:Group({})
TT5:Toggle({Title="Arrastrar (Drag)", Def=false, Callback=AS(function(s) TargetToggle("Drag", s) end)})
TargetTab:Space({Size=10})
TargetTab:Section({Title="⚡ Acciones (una vez)", TextSize=18}); TargetTab:Space({Size=6})
local TargetAct = TargetTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
local TAR1 = TargetAct:Group({})
TAR1:Button({Title="Empujar (1x)", Icon="arrow-up-right", Justify="Center", Callback=function()
    local t = TargetCurrent(); if not t then WindUI:Notify({Title="Target", Content="Sin objetivo", Duration=2}); return end
    local r = TargetGetRoot(LocalPlayer); if not r then return end
    local cf = r.CFrame
    TargetPredictionTP(t); task.wait(TargetGetPing() + 0.05); TargetPush(t); r.CFrame = cf
end})
TAR1:Space({Size=8})
TAR1:Button({Title="TP al Objetivo", Icon="map-pin", Justify="Center", Callback=function()
    local t = TargetCurrent(); if t then TargetTeleportTo(t) else WindUI:Notify({Title="Target", Content="Sin objetivo", Duration=2}) end
end})
TargetAct:Space({Size=6})
local TAR2 = TargetAct:Group({})
TAR2:Button({Title="Lista Blanca (agregar/quitar)", Icon="shield-check", Justify="Center", Callback=function()
    local t = TargetCurrent(); if not t then WindUI:Notify({Title="Target", Content="Sin objetivo", Duration=2}); return end
    local id = t.UserId; local idx = table.find(TargetWhitelist, id)
    if idx then table.remove(TargetWhitelist, idx); WindUI:Notify({Title="Target", Content=t.Name.." ELIMINADO de lista blanca", Duration=3})
    else table.insert(TargetWhitelist, id); WindUI:Notify({Title="Target", Content=t.Name.." AGREGADO a lista blanca", Duration=3}) end
end})
TAR2:Space({Size=8})
TAR2:Button({Title="Limpiar Objetivo", Icon="x", Justify="Center", Callback=function() TargetSelect(nil) end})
TargetTab:Space({Size=12})

-- 5. RJ=New.SV
local RJTab = Window:Tab({Title="RJ=New.SV", Icon="globe"})
local AutoOn = false
local AutoCoroutine = nil
local ServerList = {}
ServidoresVisitados = ServidoresVisitados or {}
local RJLimit = 1
local RJStatus, RJCounter = nil, nil
local ServerListGui = nil

local function Fetch()
    local Url = string.format("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100", game.PlaceId)
    local Ok, Data = pcall(function() return game:HttpGet(Url) end)
    if not Ok or not Data then if RJStatus and RJStatus.SetDesc then RJStatus:SetDesc("Error de conexión") end; return nil end
    local Http = game:GetService("HttpService")
    local DecodeOk, Json = pcall(Http.JSONDecode, Http, Data)
    if not DecodeOk or not Json or not Json.data then if RJStatus and RJStatus.SetDesc then RJStatus:SetDesc("Respuesta inválida") end; return nil end
    local CurrentId = tostring(game.JobId)
    local Result = {}
    for _, S in ipairs(Json.data) do
        if S.id and tostring(S.id) ~= CurrentId then
            table.insert(Result, {Id = tostring(S.id), Players = S.playing or 0, Max = S.maxPlayers or 0, Ping = S.ping or 0})
        end
    end
    return Result
end
local function JoinBest()
    local Limit = math.max(1, RJLimit)
    if #Players:GetPlayers() <= Limit then if RJStatus and RJStatus.SetDesc then RJStatus:SetDesc("Servidor óptimo") end; return end
    local Servers = Fetch() or ServerList
    if not Servers or #Servers == 0 then return end
    local Candidates = {}
    for _, S in ipairs(Servers) do if S.Players <= Limit then table.insert(Candidates, S) end end
    local Target = #Candidates > 0 and Candidates[math.random(#Candidates)] or Servers[math.random(#Servers)]
    ServidoresVisitados[Target.Id] = true; pcall(SaveConfig)
    if RJStatus and RJStatus.SetDesc then RJStatus:SetDesc("Teletransportando...") end
    pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, Target.Id, LocalPlayer) end)
end
local function SetAutoHop(on)
    AutoOn = on
    if on then
        AutoCoroutine = task.spawn(function() while AutoOn do JoinBest(); task.wait(4) end end)
    else
        if AutoCoroutine then pcall(function() task.cancel(AutoCoroutine) end); AutoCoroutine = nil end
        if RJStatus and RJStatus.SetDesc then RJStatus:SetDesc("Detenido") end
    end
end

local function CerrarListaServidores()
    if ServerListGui then pcall(function() ServerListGui:Destroy() end); ServerListGui = nil end
end
local function AbrirListaServidores()
    if ServerListGui then return end
    local Http = game:GetService("HttpService")
    local TweenService = game:GetService("TweenService")
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "ServidoresDisponibles"; ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    if gethui then ScreenGui.Parent = gethui()
    elseif syn and syn.protect_gui then syn.protect_gui(ScreenGui); ScreenGui.Parent = game.CoreGui
    else ScreenGui.Parent = game:GetService("CoreGui") end
    ServerListGui = ScreenGui
    local SelectedServer = nil; local SelectedBtn = nil
    local Main = Instance.new("Frame")
    Main.Parent = ScreenGui; Main.Name = "Main"
    Main.BackgroundColor3 = Color3.fromRGB(255, 245, 190); Main.BackgroundTransparency = 0.25; Main.BorderSizePixel = 0
    Main.Position = UDim2.new(0.5, -150, 0.5, -200); Main.Size = UDim2.new(0, 300, 0, 400)
    Main.Active = true; Main.Draggable = true; Main.ClipsDescendants = true
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
    local Stroke = Instance.new("UIStroke", Main); Stroke.Color = Color3.fromRGB(230, 200, 100); Stroke.Thickness = 1.5; Stroke.Transparency = 0.2
    local Title = Instance.new("TextLabel")
    Title.Parent = Main; Title.Position = UDim2.new(0, 12, 0, 9); Title.Size = UDim2.new(1, -100, 0, 20)
    Title.BackgroundTransparency = 1; Title.Font = Enum.Font.GothamBold; Title.Text = "Servidores Disponibles"
    Title.TextColor3 = Color3.fromRGB(110, 85, 15); Title.TextSize = 13; Title.TextXAlignment = Enum.TextXAlignment.Left
    local Refresh = Instance.new("TextButton")
    Refresh.Parent = Main; Refresh.BackgroundColor3 = Color3.fromRGB(250, 225, 130); Refresh.BackgroundTransparency = 0.10
    Refresh.Position = UDim2.new(1, -100, 0, 8); Refresh.Size = UDim2.new(0, 64, 0, 22)
    Refresh.Font = Enum.Font.GothamBold; Refresh.Text = "Recargar"; Refresh.TextColor3 = Color3.fromRGB(110, 85, 15); Refresh.TextSize = 9
    Instance.new("UICorner", Refresh).CornerRadius = UDim.new(0, 6)
    local Close = Instance.new("TextButton")
    Close.Parent = Main; Close.BackgroundColor3 = Color3.fromRGB(245, 210, 105); Close.BackgroundTransparency = 0.05
    Close.Position = UDim2.new(1, -30, 0, 8); Close.Size = UDim2.new(0, 22, 0, 22)
    Close.Font = Enum.Font.GothamBold; Close.Text = "X"; Close.TextColor3 = Color3.fromRGB(110, 80, 10); Close.TextSize = 12
    Instance.new("UICorner", Close).CornerRadius = UDim.new(0, 6)
    Close.MouseButton1Click:Connect(CerrarListaServidores)
    local List = Instance.new("ScrollingFrame")
    List.Parent = Main; List.Position = UDim2.new(0, 10, 0, 38); List.Size = UDim2.new(1, -20, 1, -90)
    List.BackgroundColor3 = Color3.fromRGB(255, 248, 210); List.BackgroundTransparency = 0.30; List.BorderSizePixel = 0
    List.ScrollBarThickness = 4; List.ScrollBarImageColor3 = Color3.fromRGB(215, 185, 80)
    List.CanvasSize = UDim2.new(0, 0, 0, 0); List.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Instance.new("UICorner", List).CornerRadius = UDim.new(0, 8)
    local Layout = Instance.new("UIListLayout"); Layout.Parent = List; Layout.Padding = UDim.new(0, 4); Layout.SortOrder = Enum.SortOrder.LayoutOrder
    local Pad = Instance.new("UIPadding"); Pad.Parent = List; Pad.PaddingTop = UDim.new(0, 6); Pad.PaddingBottom = UDim.new(0, 6)
    local function Hover(btn, normal, over)
        btn.MouseEnter:Connect(function() if not btn:GetAttribute("Selected") then TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = over}):Play() end end)
        btn.MouseLeave:Connect(function() if not btn:GetAttribute("Selected") then TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = normal}):Play() end end)
    end
    local TPBtn = Instance.new("TextButton")
    TPBtn.Parent = Main; TPBtn.Position = UDim2.new(0, 10, 1, -44); TPBtn.Size = UDim2.new(1, -20, 0, 34)
    TPBtn.BackgroundColor3 = ColorAccent; TPBtn.BackgroundTransparency = 0.05; TPBtn.BorderSizePixel = 0
    TPBtn.Font = Enum.Font.GothamBold; TPBtn.Text = "TP al Servidor Seleccionado"
    TPBtn.TextColor3 = Color3.fromRGB(90, 70, 15); TPBtn.TextSize = 12
    Instance.new("UICorner", TPBtn).CornerRadius = UDim.new(0, 8)
    RefTPBtn = TPBtn
    TPBtn.MouseButton1Click:Connect(function()
        if not SelectedServer then pcall(function() WindUI:Notify({Title="Servidores", Content="Selecciona un servidor primero", Duration=2}) end); return end
        pcall(function() ServidoresVisitados[SelectedServer.Id] = true; pcall(SaveConfig) end)
        pcall(function() WindUI:Notify({Title="Servidores", Content="Teletransportando al servidor seleccionado...", Duration=2}) end)
        pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, SelectedServer.Id, LocalPlayer) end)
    end)
    local function LoadServers()
        if not ServerListGui then return end
        SelectedServer = nil; SelectedBtn = nil
        if TPBtn then TPBtn.Text = "TP al Servidor Seleccionado" end
        for _, c in ipairs(List:GetChildren()) do if c:IsA("TextButton") or c:IsA("TextLabel") then c:Destroy() end end
        local Loading = Instance.new("TextLabel")
        Loading.Parent = List; Loading.Size = UDim2.new(1, -8, 0, 26); Loading.BackgroundTransparency = 1
        Loading.Font = Enum.Font.Gotham; Loading.Text = "Cargando servidores..."
        Loading.TextColor3 = Color3.fromRGB(110, 85, 15); Loading.TextSize = 11
        task.spawn(function()
            local url = string.format("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100", game.PlaceId)
            local ok, raw = pcall(function() return game:HttpGet(url) end)
            Loading:Destroy()
            if not ok or not raw then return end
            local decodeOk, data = pcall(Http.JSONDecode, Http, raw)
            if not decodeOk or not data or not data.data then return end
            local currentId = tostring(game.JobId)
            ServerList = {}
            for _, srv in ipairs(data.data) do
                if srv.id then
                    local esActual = tostring(srv.id) == currentId
                    if not esActual then
                        table.insert(ServerList, {Id=tostring(srv.id), Players=srv.playing or 0, Max=srv.maxPlayers or 0, Ping=srv.ping or 0})
                    end
                    local btn = Instance.new("TextButton")
                    btn.Parent = List; btn.Size = UDim2.new(1, -8, 0, 30); btn.BackgroundTransparency = 0.10
                    btn.BorderSizePixel = 0; btn.Font = Enum.Font.Gotham; btn.TextSize = 11; btn.TextXAlignment = Enum.TextXAlignment.Left
                    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
                    if esActual then
                        btn.BackgroundColor3 = Color3.fromRGB(255, 225, 90); btn.BackgroundTransparency = 0
                        btn.TextColor3 = Color3.fromRGB(110, 85, 10)
                        btn.Text = string.format("  (AQUI ESTAS) %d/%d jugadores  •  Ping: %dms", srv.playing or 0, srv.maxPlayers or 0, srv.ping or 0)
                    else
                        local visitado = ServidoresVisitados[tostring(srv.id)]
                        local baseColor = visitado and Color3.fromRGB(255, 228, 150) or Color3.fromRGB(255, 247, 200)
                        local overColor = visitado and Color3.fromRGB(250, 215, 120) or Color3.fromRGB(250, 232, 155)
                        local prefijo = visitado and "  (visitado) " or "  "
                        btn.BackgroundColor3 = baseColor; btn.TextColor3 = Color3.fromRGB(110, 85, 15)
                        btn.Text = string.format(prefijo.."%d/%d jugadores  •  Ping: %dms", srv.playing or 0, srv.maxPlayers or 0, srv.ping or 0)
                        Hover(btn, baseColor, overColor)
                        btn.MouseButton1Click:Connect(function()
                            if SelectedBtn then
                                SelectedBtn:SetAttribute("Selected", false)
                                SelectedBtn.BackgroundColor3 = Color3.fromRGB(255, 247, 200)
                                SelectedBtn.BackgroundTransparency = 0.10
                                SelectedBtn.TextColor3 = Color3.fromRGB(110, 85, 15)
                            end
                            SelectedServer = {Id=tostring(srv.id), Players=srv.playing or 0, Max=srv.maxPlayers or 0, Ping=srv.ping or 0}
                            SelectedBtn = btn
                            btn:SetAttribute("Selected", true)
                            btn.BackgroundColor3 = Color3.fromRGB(245, 205, 65)
                            btn.BackgroundTransparency = 0; btn.TextColor3 = Color3.fromRGB(90, 70, 10)
                            if TPBtn then TPBtn.Text = "TP: " .. SelectedServer.Players .. "/" .. SelectedServer.Max .. " jugadores" end
                        end)
                    end
                end
            end
        end)
    end
    Refresh.MouseButton1Click:Connect(LoadServers)
    Hover(Refresh, Color3.fromRGB(250, 225, 130), Color3.fromRGB(255, 235, 150))
    Hover(Close, Color3.fromRGB(245, 210, 105), Color3.fromRGB(255, 180, 90))
    LoadServers()
end
local function ToggleListaServidores()
    if ServerListGui then CerrarListaServidores() else AbrirListaServidores() end
end

RJTab:Section({Title="RJ = New Server", TextSize=20}); RJTab:Space({Size=6})
RJStatus = RJTab:Paragraph({Title="Estado", Desc="Listo", Image="info", ImageSize=14})
RJTab:Space({Size=4})
RJCounter = RJTab:Paragraph({Title="Jugadores", Desc="Jugadores: "..#Players:GetPlayers(), Image="users", ImageSize=14})
RJTab:Space({Size=8})
pcall(function()
    RJTab:TextBox({Title="Máx jugadores por servidor", PlaceholderText="1", Callback=function(t)
        local n = tonumber(t); if n and n >= 1 then RJLimit = math.floor(n) end
    end})
end)
RJTab:Space({Size=8})
local RJBtnRow = RJTab:Group({})
RJBtnRow:Button({Title="BUSCAR Y UNIR", Icon="send", Justify="Center", Callback=function() task.spawn(JoinBest) end})
RJBtnRow:Space({Size=8})
RJBtnRow:Toggle({Title="Auto Hop", Def=false, Callback=function(s) SetAutoHop(s) end})
RJTab:Space({Size=8})
RJTab:Button({Title="Abrir / Cerrar Lista de Servidores", Icon="globe", Justify="Center", Callback=ToggleListaServidores})
RJTab:Space({Size=12})
task.spawn(function()
    while task.wait(1) do if RJCounter and RJCounter.SetDesc then RJCounter:SetDesc("Jugadores: " .. #Players:GetPlayers()) end end
end)

-- 6. ESCUDOS
local EscudosTab = Window:Tab({Title="Escudos", Icon="shield"})
EscudosTab:Section({Title="🛡️ Protección y Defensas (reales)", TextSize=20}); EscudosTab:Space({Size=6})
EscudosTab:Paragraph({Title="Nota", Desc="Anti-Kick solo bloquea kicks de scripts locales. Un kick del servidor no se puede bloquear del lado del cliente.", Image="info", ImageSize=14})
EscudosTab:Space({Size=8})
local function shieldPair(t1, cb1, d1, t2, cb2, d2)
    local r = EscudosTab:Group({})
    r:Toggle({Title=t1, Def=d1, Callback=AS(cb1)}); r:Space({Size=8})
    r:Toggle({Title=t2, Def=d2, Callback=AS(cb2)})
    EscudosTab:Space({Size=8})
end
shieldPair("Anti-AFK (Real)", SetAntiAFK, Get("AntiAFKEnabled", false), "Anti-Kick (Hook Local)", SetAntiKick, Get("AntiKickEnabled", false))
shieldPair("Anti-Reset (Bloquea Reset)", SetAntiReset, Get("AntiResetEnabled", false), "Anti-Sit (No te sientan)", SetAntiSit, Get("AntiSitEnabled", false))
shieldPair("Anti-Fling (Anti-Lanzamiento)", SetAntiFling, Get("AntiFlingEnabled", false), "Anti-Freeze (Desanclar Auto.)", SetAntiFreeze, Get("AntiFreezeEnabled", false))
shieldPair("Anti-Ragdoll", function(s) AntiRagdollEnabled = s end, Get("AntiRagdollEnabled", false), "Anti-Void", function(s) AntiVoidEnabled = s end, Get("AntiVoidEnabled", false))
shieldPair("No Empuje", function(s) NoPushEnabled = s end, Get("NoPushEnabled", false), "No Retroceso", function(s) NoKnockbackEnabled = s end, Get("NoKnockbackEnabled", false))
shieldPair("God Mode (Local)", function(s) GodModeEnabled = s end, Get("GodModeEnabled", false), "Auto-Rejoin al Morir", function(s) AutoRejoinEnabled = s end, Get("AutoRejoinEnabled", false))
shieldPair("Anti-Report (Oculta UI)", SetAntiReport, Get("AntiReportEnabled", false), "No Fall Damage", SetNoFallDamage, Get("NoFallDamageEnabled", false))
local GAntiVC = EscudosTab:Group({})
GAntiVC:Toggle({Title="Anti-VC Ultra", Desc="Spam reconexion de voz (anti-VC agresivo)", Def=Get("AntiVCEnabled", false), Callback=AS(SetAntiVC)})
EscudosTab:Space({Size=12})

-- 7. HERRAMIENTAS
local H = Window:Tab({Title="Herramientas", Icon="wrench"})
H:Section({Title="Movimiento", TextSize=20}); H:Space({Size=6})
local Mov = H:Section({Title="Controles de Movimiento", Box=true, BoxBorder=true, Opened=true})
Mov:Toggle({Title="Fly (Volar)", Def=Get("FlyEnabled", false), Callback=AS(function(s) if s then StartFly() else StopFly() end end)}); Mov:Space({Size=6})
Mov:Slider({Title="Velocidad de Fly", Step=5, Value={Min=10,Max=200,Default=Get("FlySpeed", 60)}, Callback=AS(function(v) FlySpeed=v end)}); Mov:Space({Size=6})
Mov:Toggle({Title="Noclip", Def=Get("NoclipEnabled", false), Callback=AS(SetNoclip)}); Mov:Space({Size=6})
Mov:Toggle({Title="Salto Infinito", Def=Get("InfJumpEnabled", false), Callback=AS(SetInfJump)}); Mov:Space({Size=6})
Mov:Slider({Title="WalkSpeed", Step=1, Value={Min=16,Max=250,Default=Get("WalkSpeed", 16)}, Callback=AS(function(v) if Humanoid then Humanoid.WalkSpeed=v end end)}); Mov:Space({Size=6})
Mov:Slider({Title="JumpPower", Step=1, Value={Min=50,Max=350,Default=Get("JumpPower", 50)}, Callback=AS(function(v) if Humanoid then Humanoid.JumpPower=v end end)}); Mov:Space({Size=6})
Mov:Slider({Title="Gravedad", Step=0.1, Value={Min=0,Max=2,Default=Get("GravityScale", 1)}, Callback=AS(function(v) if Humanoid then Humanoid.GravityScale=v end end)})
H:Space({Size=10})
H:Section({Title="Visual", TextSize=20}); H:Space({Size=6})
local Vis = H:Section({Title="Efectos Visuales", Box=true, BoxBorder=true, Opened=true})
Vis:Toggle({Title="Fullbright", Def=Get("FullbrightEnabled", false), Callback=AS(SetFullbright)}); Vis:Space({Size=6})
Vis:Toggle({Title="ESP Jugadores", Def=Get("ESPEnabled", false), Callback=AS(SetESP)}); Vis:Space({Size=6})
Vis:Toggle({Title="FPS Boost", Def=Get("FpsBoostEnabled", false), Callback=AS(SetFpsBoost)}); Vis:Space({Size=6})
Vis:Slider({Title="FOV de Cámara", Step=1, Value={Min=60,Max=120,Default=Get("FOV", 70)}, Callback=AS(function(v) pcall(function() workspace.CurrentCamera.FieldOfView=v end) end)}); Vis:Space({Size=6})
Vis:Button({Title="Desbloquear Zoom", Icon="zoom-in", Justify="Center", Callback=function() pcall(function() workspace.CurrentCamera.CameraMaxZoomDistance=1000; workspace.CurrentCamera.CameraMinZoomDistance=0.5 end); WindUI:Notify({Title="Zoom", Content="Zoom desbloqueado", Duration=2}) end})
H:Space({Size=10})
H:Section({Title="Utilidades", TextSize=20}); H:Space({Size=6})
local Uti = H:Section({Title="Herramientas Universales", Box=true, BoxBorder=true, Opened=true})
Uti:Toggle({Title="Auto-Rejoin al Morir", Def=Get("AutoRejoinEnabled", false), Callback=AS(function(s) AutoRejoinEnabled=s end)}); Uti:Space({Size=6})
Uti:Toggle({Title="Anti-Void", Def=Get("AntiVoidEnabled", false), Callback=AS(function(s) AntiVoidEnabled=s end)}); Uti:Space({Size=6})
Uti:Toggle({Title="Anti-Ragdoll", Def=Get("AntiRagdollEnabled", false), Callback=AS(function(s) AntiRagdollEnabled=s end)}); Uti:Space({Size=8})
SpectateDropdown = Uti:Dropdown({Title="Jugador a Espectear", Values=GetPlayerNames(true), Value=1, Callback=function(s) SpectateName=s end})
Uti:Space({Size=6})
local SpR=Uti:Group({})
SpR:Button({Title="Espectear", Icon="eye", Justify="Center", Callback=function()
    if not SpectateName or SpectateName=="Nadie (detener)" or SpectateName=="No hay jugadores" then pcall(function() workspace.CurrentCamera.CameraSubject=Humanoid end); WindUI:Notify({Title="Espectear", Content="Selecciona un jugador", Duration=2}); return end
    local t=Players:FindFirstChild(SpectateName)
    if t and t.Character and t.Character:FindFirstChild("Humanoid") then workspace.CurrentCamera.CameraSubject=t.Character.Humanoid; WindUI:Notify({Title="Especteando", Content="A "..SpectateName, Duration=2})
    else WindUI:Notify({Title="Error", Content="Jugador no disponible", Duration=2}) end
end})
SpR:Space({Size=8})
SpR:Button({Title="Detener", Icon="eye-off", Justify="Center", Callback=function() pcall(function() workspace.CurrentCamera.CameraSubject=Humanoid end); WindUI:Notify({Title="Espectear", Content="Detenido", Duration=2}) end})
Uti:Space({Size=8})
TPPlayerDropdown = Uti:Dropdown({Title="TP a Jugador", Values=GetPlayerNames(false), Value=1, Callback=function(s) TPPlayerName=s end})
Uti:Space({Size=6})
Uti:Button({Title="Teletransportar a Jugador", Icon="map-pin", Justify="Center", Callback=function()
    if not TPPlayerName or TPPlayerName=="No hay jugadores" then WindUI:Notify({Title="TP", Content="Selecciona un jugador", Duration=2}); return end
    local t=Players:FindFirstChild(TPPlayerName)
    if t and t.Character and t.Character:FindFirstChild("HumanoidRootPart") and RootPart then RootPart.CFrame=t.Character.HumanoidRootPart.CFrame*CFrame.new(0,0,4); WindUI:Notify({Title="TP", Content="A "..TPPlayerName, Duration=2})
    else WindUI:Notify({Title="Error", Content="Jugador no disponible", Duration=2}) end
end})
Uti:Space({Size=6})
Uti:Button({Title="Recargar Listas de Jugadores", Icon="refresh-cw", Justify="Center", Callback=function()
    pcall(function() SpectateDropdown:Refresh(GetPlayerNames(true)) end)
    pcall(function() TPPlayerDropdown:Refresh(GetPlayerNames(false)) end)
    pcall(function() if TargetDropdown then TargetDropdown:Refresh(GetPlayerNames(false)) end end)
    WindUI:Notify({Title="Listas", Content="Jugadores recargados", Duration=2})
end})
Uti:Space({Size=8})
Uti:Button({Title="Server Hop", Icon="shuffle", Justify="Center", Callback=ServerHop}); Uti:Space({Size=6})
local UB1=Uti:Group({})
UB1:Button({Title="Copiar Link del Juego", Icon="link", Justify="Center", Callback=function() setclipboard("https://www.roblox.com/games/"..tostring(game.PlaceId)); WindUI:Notify({Title="Copiado", Content="Link copiado", Duration=2}) end})
UB1:Space({Size=8})
UB1:Button({Title="Copiar Link del Servidor", Icon="link-2", Justify="Center", Callback=function() setclipboard("roblox://placeId="..tostring(game.PlaceId).."&jobId="..tostring(game.JobId)); WindUI:Notify({Title="Copiado", Content="Link copiado", Duration=2}) end})
Uti:Space({Size=6})
local UB2=Uti:Group({})
UB2:Button({Title="Rejoin", Icon="refresh-cw", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end})
UB2:Space({Size=8})
UB2:Button({Title="Volver al Spawn", Icon="home", Justify="Center", Callback=function() if RootPart then RootPart.CFrame=SpawnCFrame; RootPart.Velocity=Vector3.new(0,0,0); WindUI:Notify({Title="Spawn", Content="Volviste", Duration=2}) end end})
Uti:Space({Size=8})
CoordsParagraph = Uti:Paragraph({Title="Coordenadas Actuales", Desc="X: 0  Y: 0  Z: 0", Image="map", ImageSize=14})
H:Space({Size=10})
H:Section({Title="Movimiento Extra", TextSize=20}); H:Space({Size=6})
local ME=H:Section({Title="Movimiento Adicional", Box=true, BoxBorder=true, Opened=true})
ME:Toggle({Title="Click TP (Clic Derecho)", Def=Get("ClickTPEnabled", false), Callback=AS(SetClickTP)}); ME:Space({Size=6})
ME:Toggle({Title="Auto-Jump (Bunny Hop)", Def=Get("AutoJumpEnabled", false), Callback=AS(function(s) AutoJumpEnabled=s end)}); ME:Space({Size=6})
ME:Toggle({Title="Walk on Water", Def=Get("WalkOnWaterEnabled", false), Callback=AS(function(s) WalkOnWaterEnabled=s end)}); ME:Space({Size=6})
ME:Toggle({Title="Spin Bot", Def=Get("SpinBotEnabled", false), Callback=AS(SetSpinBot)}); ME:Space({Size=6})
ME:Toggle({Title="Freeze Position", Def=Get("FreezePositionEnabled", false), Callback=AS(SetFreeze)}); ME:Space({Size=6})
ME:Slider({Title="Velocidad de Caída Máx.", Step=10, Value={Min=10,Max=200,Default=Get("FallSpeedCap", 200)}, Callback=AS(function(v) FallSpeedCap=v end)}); ME:Space({Size=6})
ME:Slider({Title="Tamaño de Personaje", Step=0.1, Value={Min=0.3,Max=5,Default=Get("BodyScale", 1)}, Callback=AS(function(v) pcall(function() if Humanoid then Humanoid.BodyHeightScale=v; Humanoid.BodyWidthScale=v; Humanoid.BodyDepthScale=v; if Humanoid.HeadScale then Humanoid.HeadScale=v end end end) end)})
H:Space({Size=10})
H:Section({Title="Visual Extra", TextSize=20}); H:Space({Size=6})
local VE=H:Section({Title="Efectos Adicionales", Box=true, BoxBorder=true, Opened=true})
VE:Slider({Title="Velocidad de Animación", Step=0.1, Value={Min=0.1,Max=5,Default=Get("AnimationSpeed", 1)}, Callback=AS(function(v) if Humanoid then pcall(function() Humanoid.AnimationSpeed=v end) end end)}); VE:Space({Size=6})
VE:Slider({Title="Hora del Día (ClockTime)", Step=1, Value={Min=0,Max=24,Default=Get("ClockTime", 14)}, Callback=AS(function(v) Lighting.ClockTime=v end)}); VE:Space({Size=6})
VE:Button({Title="Eliminar Partículas/Efectos", Icon="trash-2", Justify="Center", Callback=RemoveParticles})
H:Space({Size=10})
H:Section({Title="Utilidades Extra", TextSize=20}); H:Space({Size=6})
local UE=H:Section({Title="Herramientas Adicionales", Box=true, BoxBorder=true, Opened=true})
UE:Toggle({Title="God Mode (Local)", Def=Get("GodModeEnabled", false), Callback=AS(function(s) GodModeEnabled=s end)}); UE:Space({Size=6})
UE:Toggle({Title="Auto-Respawn Instantáneo", Def=Get("InstantRespawnEnabled", false), Callback=AS(function(s) InstantRespawnEnabled=s end)}); UE:Space({Size=6})
UE:Toggle({Title="Seguir Jugador (Follow)", Def=Get("FollowPlayerEnabled", false), Callback=AS(function(s) FollowPlayerEnabled=s end)}); UE:Space({Size=8})
pcall(function() UE:TextBox({Title="Coordenadas (X, Y, Z)", PlaceholderText="0, 10, 0", Callback=function(t) CoordsText=t end}) end)
UE:Space({Size=6})
UE:Button({Title="TP a Coordenadas", Icon="map-pin", Justify="Center", Callback=function() TeleportToCoords(CoordsText) end}); UE:Space({Size=6})
local SvR=UE:Group({})
SvR:Button({Title="Guardar Posición", Icon="save", Justify="Center", Callback=function() if RootPart then SavedPosition=RootPart.CFrame end; WindUI:Notify({Title="Guardado", Content="Posición guardada", Duration=2}) end})
SvR:Space({Size=8})
SvR:Button({Title="Volver a Posición", Icon="home", Justify="Center", Callback=function() if SavedPosition and RootPart then RootPart.CFrame=SavedPosition; RootPart.Velocity=Vector3.new(0,0,0); WindUI:Notify({Title="TP", Content="Volviste", Duration=2}) else WindUI:Notify({Title="Error", Content="No hay posición guardada", Duration=2}) end end})
H:Space({Size=12})

-- 8. CRÉDITOS
local Cr = Window:Tab({Title="Créditos", Icon="award"})
Cr:Section({Title="Agradecimientos", TextSize=20}); Cr:Space({Size=6})
local CG=Cr:Group({})
CG:Paragraph({Title="Creador", Desc="Takemichi_01x\n© 2026", Image="code", ImageSize=16}); CG:Space({Size=10})
CG:Paragraph({Title="UI Library", Desc="WindUI v1.6.65\nFootagesus", Image="book", ImageSize=16})
Cr:Space({Size=8})
Cr:Paragraph({Title="Gracias por usar", Desc="¡Disfruta el script!", Image="heart", ImageSize=16})
Cr:Space({Size=12})
Cr:Paragraph({Title="", Desc="tonto el que ha leído esto", Image="smile", ImageSize=16})

-- LOOP EN TIEMPO REAL
task.spawn(function()
    while task.wait(0.1) do
        if not Character or not RootPart then UpdateChar() end
        if NoFrictionEnabled and RootPart then RootPart.Friction=0; RootPart.AirFriction=0
        elseif RootPart then RootPart.Friction=1; RootPart.AirFriction=0.5 end
        if AntiFlingEnabled and RootPart then
            RootPart.CustomPhysicalProperties = PhysicalProperties.new(100, NoPushEnabled and 0 or 1, 0)
        elseif (NoPushEnabled or NoKnockbackEnabled) and RootPart then
            RootPart.CustomPhysicalProperties=PhysicalProperties.new(10, 0.3, NoPushEnabled and 0 or 0.5)
        elseif RootPart then
            RootPart.CustomPhysicalProperties=PhysicalProperties.new(1, 0.5, 0.5)
        end
        if AntiVoidEnabled and RootPart then
            if RootPart.Position.Y>-10 then LastSafePos=RootPart.Position
            elseif RootPart.Position.Y<-50 then RootPart.CFrame=CFrame.new(LastSafePos+Vector3.new(0,5,0)); RootPart.Velocity=Vector3.new(0,0,0) end
        end
        if AntiRagdollEnabled and Humanoid then
            local st=Humanoid:GetState()
            if st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown or st==Enum.HumanoidStateType.Physics then Humanoid:ChangeState(Enum.HumanoidStateType.GettingUp) end
        end
        if GodModeEnabled and Humanoid then pcall(function() Humanoid.Health=Humanoid.MaxHealth end) end
        if AutoJumpEnabled and Humanoid then pcall(function() if Humanoid.FloorMaterial~=Enum.Material.Air then Humanoid.Jump=true end end) end
        if WalkOnWaterEnabled and Humanoid and RootPart then pcall(function()
            if Humanoid:GetState()==Enum.HumanoidStateType.Swimming then Humanoid:ChangeState(Enum.HumanoidStateType.Running); RootPart.Velocity=Vector3.new(RootPart.Velocity.X, 50, RootPart.Velocity.Z) end
        end) end
        if RootPart and RootPart.Velocity.Y<-FallSpeedCap then RootPart.Velocity=Vector3.new(RootPart.Velocity.X, -FallSpeedCap, RootPart.Velocity.Z) end
        if FollowPlayerEnabled and RootPart then
            local t=TPPlayerName and Players:FindFirstChild(TPPlayerName)
            if t and t.Character and t.Character:FindFirstChild("HumanoidRootPart") then RootPart.CFrame=t.Character.HumanoidRootPart.CFrame*CFrame.new(0,0,5) end
        end
        if AntiSitEnabled and Humanoid then
            pcall(function() Humanoid.Sit=false; Humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, false) end)
        end
        if AntiFreezeEnabled and not FreezePositionEnabled and RootPart and RootPart.Anchored then RootPart.Anchored = false end
        if CoordsParagraph and RootPart and CoordsParagraph.SetDesc then
            local p=RootPart.Position; CoordsParagraph:SetDesc("X: "..math.floor(p.X).."  Y: "..math.floor(p.Y).."  Z: "..math.floor(p.Z))
        end
    end
end)

local function HookDeath(char)
    local hum=char:WaitForChild("Humanoid", 10)
    if hum then hum.Died:Connect(function()
        if InstantRespawnEnabled then task.wait(0.1); pcall(function() LocalPlayer:LoadCharacter() end); return end
        if AutoRejoinEnabled then task.wait(3); pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end) end
    end) end
end
if Character then HookDeath(Character) end
LocalPlayer.CharacterAdded:Connect(HookDeath)

pcall(function()
    local SideBar=Window.SideBar
    if SideBar and SideBar.Container then
        local layout=SideBar.Container:FindFirstChildOfClass("UIListLayout")
        if layout then layout.Padding=UDim.new(0,12); layout.Spacing=UDim.new(0,18) end
    end
end)

AplicarConfiguracion()
task.spawn(function() task.wait(1.5); pcall(AplicarConfiguracion) end)

-- CUADRO DE PERFIL
task.spawn(function()
    pcall(function()
        task.wait(0.6)
        if not PlayerTab then return end
        local Contenedor = nil
        pcall(function() Contenedor = PlayerTab.UIElements and PlayerTab.UIElements.ContainerFrame end)
        if not Contenedor then pcall(function() Contenedor = PlayerTab.ContainerFrame end) end)
        if not Contenedor then pcall(function() Contenedor = PlayerTab.Container end) end)
        if not Contenedor then pcall(function() Contenedor = Window.SideBar and Window.SideBar.Parent end) end)
        if not Contenedor then return end
        local Box = Instance.new("Frame")
        Box.Name = "PerfilCuadro"; Box.Parent = Contenedor; Box.LayoutOrder = -100
        Box.Size = UDim2.new(1, -16, 0, 116)
        Box.BackgroundColor3 = Color3.fromRGB(255, 255, 255); Box.BackgroundTransparency = 0.86
        Box.BorderSizePixel = 0; Box.Active = false
        Instance.new("UICorner", Box).CornerRadius = UDim.new(0, 10)
        local BordeBox = Instance.new("UIStroke"); BordeBox.Parent = Box; BordeBox.Thickness = 1; BordeBox.Color = ColorAccent; BordeBox.Transparency = 0.35
        RefBordePerfil = BordeBox
        local Circulo = Instance.new("Frame")
        Circulo.Name = "CirculoPerfil"; Circulo.Parent = Box
        Circulo.BackgroundColor3 = Color3.fromRGB(255, 240, 170); Circulo.BackgroundTransparency = 1
        Circulo.Position = UDim2.new(1, -108, 0.5, -50); Circulo.Size = UDim2.new(0, 100, 0, 100)
        Circulo.ZIndex = 51; Circulo.Active = false
        Instance.new("UICorner", Circulo).CornerRadius = UDim.new(1, 0)
        local Borde = Instance.new("UIStroke"); Borde.Thickness = 3; Borde.Color = ColorAccent; Borde.Parent = Circulo
        RefBordeCirculo = Borde
        local Foto = Instance.new("ImageLabel")
        Foto.Name = "Foto"; Foto.Parent = Circulo; Foto.BackgroundTransparency = 1
        Foto.Position = UDim2.new(0, 5, 0, 5); Foto.Size = UDim2.new(1, -10, 1, -10); Foto.ZIndex = 52
        Instance.new("UICorner", Foto).CornerRadius = UDim.new(1, 0)
        local Cargar = pcall(function() Foto.Image = Players:GetUserThumbnailAsync(UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420) end)
        if not Cargar then Foto.Image = "rbxassetid://6026588573" end
        local function HacerTexto(y, texto, tamano, bold, color)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = Box; lbl.BackgroundTransparency = 1
            lbl.Position = UDim2.new(0, 14, 0, y); lbl.Size = UDim2.new(1, -130, 0, tamano + 6)
            lbl.Font = bold and Enum.Font.GothamBold or Enum.Font.Gotham
            lbl.Text = texto; lbl.TextColor3 = color; lbl.TextSize = tamano
            lbl.TextXAlignment = Enum.TextXAlignment.Left; lbl.TextTruncate = Enum.TextTruncate.AtEnd
            return lbl
        end
        HacerTexto(14, DisplayName, 19, true, Color3.fromRGB(245, 245, 250))
        HacerTexto(44, "@"..PlayerName, 14, false, Color3.fromRGB(245, 245, 250))
        HacerTexto(68, "ID: "..tostring(UserId), 14, false, Color3.fromRGB(245, 245, 250))
    end)
end)

pcall(function() Window:SelectTab(PlayerTab) end)
pcall(function() Window:SelectTab(1) end)

WindUI:Notify({Title="CHOCOFRZ🍰", Content="v34: Tema Amarillo Pastel + foto de fondo más clara", Duration=4})

-- ==============================================================
-- [v35] PARCHE FINAL CORREGIDO: Repintar + aclarar fondo
-- ==============================================================
task.spawn(function()
    task.wait(0.4)

    -- Obtener la GUI principal de WindUI
    local gui = nil
    pcall(function() gui = Window.Gui end)
    if not gui then
        pcall(function()
            for _, g in ipairs(game:GetService("CoreGui"):GetChildren()) do
                if g:IsA("ScreenGui") and (g.Name:lower():find("wind") or g.Name:lower():find("ui")) then gui = g; break end
            end
        end)
    end
    if not gui then
        pcall(function()
            for _, g in ipairs(LocalPlayer:WaitForChild("PlayerGui"):GetChildren()) do
                if g:IsA("ScreenGui") and (g.Name:lower():find("wind") or g.Name:lower():find("ui")) then gui = g; break end
            end
        end)
    end
    if not gui then return end

    local Amarillo       = Color3.fromRGB(255, 235, 140)
    local AmarilloVivo   = Color3.fromRGB(245, 205, 65)
    local ColorTextoOscuro = Color3.fromRGB(90, 70, 15)

    -- Pintar botones
    local function PintarBoton(d)
        if not (d:IsA("TextButton") or d:IsA("ImageButton")) then return end
        pcall(function()
            local c = d.BackgroundColor3
            if c.R < 0.5 and c.G < 0.5 and c.B < 0.5 then
                d.BackgroundColor3 = Amarillo
                if d.BackgroundTransparency > 0.5 then d.BackgroundTransparency = 0.15 end
            end
        end)
    end

    -- Pintar knob de sliders / toggles (círculos pequeños)
    local function PintarKnob(d)
        if not d:IsA("Frame") then return end
        pcall(function()
            local corner = d:FindFirstChildOfClass("UICorner")
            if corner and corner.CornerRadius.Scale >= 0.9 then
                local sz = d.AbsoluteSize
                if sz.X > 8 and sz.X < 45 and sz.Y > 8 and sz.Y < 45 then
                    d.BackgroundColor3 = AmarilloVivo
                end
            end
        end)
    end

    -- Aclarar foto de fondo y velo oscuro
    local function AclararFondo()
        pcall(function()
            for _, d in ipairs(gui:GetDescendants()) do
                if d:IsA("ImageLabel") and d.Image and d.Image:find("81943489858207") then
                    d.ImageTransparency = 0.82
                end
                if d:IsA("Frame") and not d:FindFirstAncestorWhichIsA("TextButton") then
                    local tamX, tamY = d.AbsoluteSize.X, d.AbsoluteSize.Y
                    if tamX > 280 and tamY > 180 and d.BackgroundTransparency < 0.95 then
                        local c = d.BackgroundColor3
                        if c.R < 0.42 and c.G < 0.42 and c.B < 0.42 then
                            d.BackgroundColor3 = Color3.fromRGB(82, 66, 35)
                            d.BackgroundTransparency = 0.50
                        end
                    end
                end
            end
        end)
    end

    local function RepintarTodo()
        pcall(function()
            for _, d in ipairs(gui:GetDescendants()) do
                PintarBoton(d)
                PintarKnob(d)
            end
        end)
    end

    AclararFondo()
    RepintarTodo()

    -- Texto del cuadro de perfil (legible sobre fondo claro)
    pcall(function()
        local box = gui:FindFirstChild("PerfilCuadro", true)
        if box then
            for _, lbl in ipairs(box:GetDescendants()) do
                if lbl:IsA("TextLabel") then lbl.TextColor3 = ColorTextoOscuro end
            end
        end
    end)

    -- Repintar elementos que aparecen después (dropdowns, tabs, etc.)
    gui.DescendantAdded:Connect(function(d)
        task.wait(0.08)
        PintarBoton(d)
        PintarKnob(d)
    end)

    -- Re-afirmar cada segundo por si WindUI repinta al cambiar de pestaña
    while task.wait(1) do
        if not gui or not gui.Parent then break end
        RepintarTodo()
    end
end)

pcall(function()
    WindUI:Notify({Title="TKSH v35", Content="Foto aclarada + botones amarillo pastel", Duration=4})
end)
