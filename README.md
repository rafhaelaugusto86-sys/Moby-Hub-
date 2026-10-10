local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local UI_LIB_URL = "https://pastebin.com/raw/WMpvdEDV"
local function LoadUILib()
local urls = {
UI_LIB_URL,
"https://pastebin.com/raw/WMpvdEDV"
}
for _, url in ipairs(urls) do
if url and url ~= "" then
local success, source = pcall(game.HttpGet, game, url)
if success and source and #source > 0 and not source:find("404: Not Found") then
local fn, err = loadstring(source)
if fn then
return fn()
end
end
end
end
error("Failed to load UI library from " .. tostring(UI_LIB_URL))
end
local library = LoadUILib()
library.title = "Moby Hub"
local Init = library:Init(Enum.KeyCode.LeftAlt)
local dummyNotif = {}
function dummyNotif:Text() return dummyNotif end
function dummyNotif:Hide() return dummyNotif end
function dummyNotif:Show() return dummyNotif end
function dummyNotif:Remove() return dummyNotif end
local Notif = library:InitNotifications()
local _origNotify = Notif.Notify
function Notif:Notify(text, duration, notifType, callback)
if (State and State.DisableNotifications) or (library and library.DisableNotifications) or (getgenv and getgenv().DisableNotifications) then
return dummyNotif
end
return _origNotify(self, text, duration, notifType, callback)
end
local replicatedStorage = game:GetService("ReplicatedStorage")
local eventsFolder = replicatedStorage:FindFirstChild("Events")
local localRoleFromEvent = nil
if eventsFolder then
local remoteEvents = eventsFolder:FindFirstChild("RemoteEvents")
if remoteEvents then
local showRole = remoteEvents:FindFirstChild("showRole")
if showRole and showRole:IsA("RemoteEvent") then
showRole.OnClientEvent:Connect(function(role, time)
local rStr = tostring(role):lower()
local displayRole = tostring(role)
if rStr:find("murder") or rStr:find("bad") then
displayRole = "Bad Guy"
localRoleFromEvent = "Murderer"
elseif rStr:find("sherif") or rStr:find("protect") then
displayRole = "Protector"
localRoleFromEvent = "Sheriff"
elseif rStr:find("inocent") or rStr:find("innocent") or rStr:find("good") then
displayRole = "Good Guy"
localRoleFromEvent = "Innocent"
end
local dur = (type(time) == "number" and time > 0) and time or 6
Notif:Notify("Role: " .. displayRole, dur, "information")
end)
end
end
end
LocalPlayer.CharacterAdded:Connect(function()
localRoleFromEvent = nil
end)
if not UserInputService.TouchEnabled then
local Wm = library:Watermark("Moby Hub")
local FpsWm = Wm:AddWatermark("fps: " .. library.fps)
local PingWm = Wm:AddWatermark("ping: 0 ms")
task.spawn(function()
local StatsService = game:GetService("Stats")
while task.wait(1) do
FpsWm:Text("fps: " .. library.fps)
local ping = 0
pcall(function()
ping = math.floor(StatsService.Network.ServerStatsItem["Data Ping"]:GetValue())
end)
if ping == 0 then
pcall(function()
ping = math.floor(LocalPlayer:GetNetworkPing() * 1000)
end)
end
PingWm:Text("ping: " .. tostring(ping) .. " ms")
end
end)
end
local State = {
DisableNotifications = false,
Chams = false,
Nametags = false,
ChamsObjects = {},
NametagObjects = {},
SelectedTargets = {},
TargetMode = "Selected",
FlingActive = false,
FlingTargetName = nil,
TouchFling = false,
Noclip = false,
InfJump = false,
InfJumpConn = nil,
WalkSpeedEnabled = false,
WalkSpeed = 16,
OriginalWalkSpeed = 16,
TPWalk = false,
TPWalkSpeed = 3,
Bhop = false,
BhopSpeed = 50,
Shaders = false,
SilentAim = false,
AutoCoins = false,
AutoKillAll = false,
AutoGrabGun = false,
AutoShootBadGuy = false,
ShootPrediction = true,
PredictionAmount = 100,
PingCompensation = true,
MoveDirPrediction = true,
PredictYAxis = true,
WallCheck = true,
HitboxTarget = "HumanoidRootPart",
AntiAimResolver = true,
ResolverMode = "Delta Calculation",
ResolverMaxVel = 65,
AntiAfk = false,
AutoSafeTeleport = false,
SafeTeleportPos = Vector3.new(0, 505, 0),
AntiSit = false,
AntiFling = false,
AutoRemoveCars = false,
AntiGameplayPaused = false,
SafePlatform = nil,
CtrlClickTp = false,
CtrlTpConn = nil,
CurrentConfigName = "default",
SelectedConfigName = "default",
ConfigDropdown = nil,
Toggles = {},
Sliders = {},
Keybinds = {},
Selectors = {},
Autoload = false,
AutoloadToggle = nil
}
getgenv().OldPos = nil
getgenv().FPDH = workspace.FallenPartsDestroyHeight
local Mouse = LocalPlayer:GetMouse()
State.CtrlTpConn = Mouse.Button1Down:Connect(function()
if State.CtrlClickTp and (UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.RightControl)) then
if Mouse.Target and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Mouse.Hit.Position + Vector3.new(0, 3, 0))
end
end
end)
local function clearTable(t)
for k in pairs(t) do
t[k] = nil
end
end
local function GetRoleColor(player)
local char = player.Character
local function checkAttr(name)
if player:GetAttribute(name) == true then return true end
if char and char:GetAttribute(name) == true then return true end
return false
end
if checkAttr("murderer") or checkAttr("Murderer") or checkAttr("murder") then
return Color3.fromRGB(255, 0, 0)
elseif checkAttr("innocent") or checkAttr("Innocent") or checkAttr("inocent") then
return Color3.fromRGB(0, 255, 0)
elseif checkAttr("sheriff") or checkAttr("Sheriff") or checkAttr("sherif") then
return Color3.fromRGB(0, 150, 255)
end
return Color3.fromRGB(128, 128, 128)
end
local function RemovePlayerNametag(player)
if State.NametagObjects[player] then
pcall(function() State.NametagObjects[player]:Destroy() end)
State.NametagObjects[player] = nil
end
end
local function GetPlayerNametag(player, head)
if not State.NametagObjects[player] or not State.NametagObjects[player].Parent then
local bg = Instance.new("BillboardGui")
bg.Name = "MM2KidsNametag"
bg.Adornee = head
bg.Size = UDim2.new(0, 160, 0, 24)
bg.StudsOffset = Vector3.new(0, 1.8, 0)
bg.AlwaysOnTop = true
bg.ResetOnSpawn = false
bg.Parent = CoreGui
local label = Instance.new("TextLabel")
label.Name = "TagLabel"
label.Parent = bg
label.BackgroundTransparency = 1
label.Size = UDim2.new(1, 0, 1, 0)
label.Font = Enum.Font.Code
label.TextSize = 13
label.TextColor3 = Color3.fromRGB(255, 255, 255)
label.TextStrokeTransparency = 0
label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
label.Text = player.DisplayName or player.Name
State.NametagObjects[player] = bg
end
return State.NametagObjects[player]
end
local function UpdateVisuals()
for _, player in ipairs(Players:GetPlayers()) do
if player ~= LocalPlayer then
local char = player.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
local head = char and (char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart"))
local isAlive = char and hum and hum.Health > 0 and head
if State.Chams and isAlive then
if not State.ChamsObjects[player] or not State.ChamsObjects[player].Parent then
local highlight = Instance.new("Highlight")
highlight.Name = "MM2KidsChams"
highlight.OutlineTransparency = 1
highlight.FillTransparency = 0.5
highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
highlight.Parent = CoreGui
State.ChamsObjects[player] = highlight
end
if State.ChamsObjects[player] then
State.ChamsObjects[player].Adornee = char
local newColor = GetRoleColor(player)
if State.ChamsObjects[player].FillColor ~= newColor then
State.ChamsObjects[player].FillColor = newColor
State.ChamsObjects[player].Enabled = false
State.ChamsObjects[player].Enabled = true
end
end
else
if State.ChamsObjects[player] then
pcall(function() State.ChamsObjects[player]:Destroy() end)
State.ChamsObjects[player] = nil
end
end
if State.Nametags and isAlive then
local bg = GetPlayerNametag(player, head)
bg.Adornee = head
bg.Enabled = true
local label = bg:FindFirstChild("TagLabel")
if label then
label.Text = player.DisplayName or player.Name
label.TextColor3 = GetRoleColor(player)
end
else
RemovePlayerNametag(player)
end
end
end
end
UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
if not gameProcessed and input.KeyCode == Enum.KeyCode.LeftAlt then
for _, gui in pairs(CoreGui:GetChildren()) do
if gui:IsA("ScreenGui") and (gui.Name == "UnLeaked" or gui:FindFirstChild("Main") or gui.Name:match("selfcode")) then
gui.Enabled = not gui.Enabled
end
end
end
end)
LocalPlayer.Idled:Connect(function()
if State.AntiAfk then
VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
task.wait(1)
VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
end
end)
local Lighting = game:GetService("Lighting")
local shaderInstances = {}
local originalLightingProps = {}
local ShaderPresets = {
night = {
Ambient = Color3.fromRGB(33, 33, 33),
Atmosphere_Color = Color3.fromRGB(175, 221, 255),
Atmosphere_Decay = Color3.fromRGB(13, 105, 172),
Atmosphere_Density = 0.264,
Atmosphere_Glare = 0.36,
Atmosphere_Haze = 1.72,
Atmosphere_Offset = 0.156,
Bloom_Intensity = 0.34,
Bloom_Size = 10.0,
Bloom_Threshold = 0.813,
Blur_Size = 5.0,
Brightness = 3.25,
ClockTime = 20.0,
Clouds_Color = Color3.fromRGB(255, 255, 255),
Clouds_Cover = 0.65,
Clouds_Density = 0.33,
ColorCorrection_Brightness = -0.06,
ColorCorrection_Contrast = -0.02,
ColorCorrection_Saturation = -0.2,
ColorCorrection_TintColor = Color3.fromRGB(242, 243, 243),
ColorShift_Bottom = Color3.fromRGB(0, 0, 0),
ColorShift_Top = Color3.fromRGB(255, 247, 237),
DepthOfField_FarIntensity = 0.217,
DepthOfField_FocusDistance = 11.54,
DepthOfField_InFocusRadius = 16.77,
DepthOfField_NearIntensity = 0.277,
EnvironmentDiffuseScale = 0.203,
EnvironmentSpecularScale = 0.255,
ExposureCompensation = 0.85,
GeographicLatitude = -15.0,
GlobalShadows = true,
OutdoorAmbient = Color3.fromRGB(51, 54, 67)
}
}
local function UpdateShaders(enabled)
State.Shaders = enabled
for _, inst in ipairs(shaderInstances) do
pcall(function() inst:Destroy() end)
end
shaderInstances = {}
if enabled then
originalLightingProps = {
Ambient = Lighting.Ambient,
Brightness = Lighting.Brightness,
ClockTime = Lighting.ClockTime,
ColorShift_Bottom = Lighting.ColorShift_Bottom,
ColorShift_Top = Lighting.ColorShift_Top,
EnvironmentDiffuseScale = Lighting.EnvironmentDiffuseScale,
EnvironmentSpecularScale = Lighting.EnvironmentSpecularScale,
ExposureCompensation = Lighting.ExposureCompensation,
GeographicLatitude = Lighting.GeographicLatitude,
GlobalShadows = Lighting.GlobalShadows,
OutdoorAmbient = Lighting.OutdoorAmbient
}
local p = ShaderPresets.night
pcall(function()
Lighting.Ambient = p.Ambient
Lighting.Brightness = p.Brightness
Lighting.ClockTime = p.ClockTime
Lighting.ColorShift_Bottom = p.ColorShift_Bottom
Lighting.ColorShift_Top = p.ColorShift_Top
Lighting.EnvironmentDiffuseScale = p.EnvironmentDiffuseScale
Lighting.EnvironmentSpecularScale = p.EnvironmentSpecularScale
Lighting.ExposureCompensation = p.ExposureCompensation
Lighting.GeographicLatitude = p.GeographicLatitude
Lighting.GlobalShadows = p.GlobalShadows
Lighting.OutdoorAmbient = p.OutdoorAmbient
end)
local atmosphere = Instance.new("Atmosphere")
atmosphere.Name = "MM2KidsAtmosphere"
atmosphere.Color = p.Atmosphere_Color
atmosphere.Decay = p.Atmosphere_Decay
atmosphere.Density = p.Atmosphere_Density
atmosphere.Glare = p.Atmosphere_Glare
atmosphere.Haze = p.Atmosphere_Haze
atmosphere.Offset = p.Atmosphere_Offset
atmosphere.Parent = Lighting
table.insert(shaderInstances, atmosphere)
local bloom = Instance.new("BloomEffect")
bloom.Name = "MM2KidsBloom"
bloom.Intensity = p.Bloom_Intensity
bloom.Size = p.Bloom_Size
bloom.Threshold = p.Bloom_Threshold
bloom.Parent = Lighting
table.insert(shaderInstances, bloom)
local blur = Instance.new("BlurEffect")
blur.Name = "MM2KidsBlur"
blur.Size = p.Blur_Size
blur.Parent = Lighting
table.insert(shaderInstances, blur)
local cc = Instance.new("ColorCorrectionEffect")
cc.Name = "MM2KidsColorCorrection"
cc.Brightness = p.ColorCorrection_Brightness
cc.Contrast = p.ColorCorrection_Contrast
cc.Saturation = p.ColorCorrection_Saturation
cc.TintColor = p.ColorCorrection_TintColor
cc.Parent = Lighting
table.insert(shaderInstances, cc)
local dof = Instance.new("DepthOfFieldEffect")
dof.Name = "MM2KidsDepthOfField"
dof.FarIntensity = p.DepthOfField_FarIntensity
dof.FocusDistance = p.DepthOfField_FocusDistance
dof.InFocusRadius = p.DepthOfField_InFocusRadius
dof.NearIntensity = p.DepthOfField_NearIntensity
dof.Parent = Lighting
table.insert(shaderInstances, dof)
local terrain = workspace:FindFirstChildOfClass("Terrain")
if terrain then
local clouds = Instance.new("Clouds")
clouds.Name = "MM2KidsClouds"
clouds.Color = p.Clouds_Color
clouds.Cover = p.Clouds_Cover
clouds.Density = p.Clouds_Density
clouds.Parent = terrain
table.insert(shaderInstances, clouds)
end
else
pcall(function()
if originalLightingProps.Ambient then
Lighting.Ambient = originalLightingProps.Ambient
Lighting.Brightness = originalLightingProps.Brightness
Lighting.ClockTime = originalLightingProps.ClockTime
Lighting.ColorShift_Bottom = originalLightingProps.ColorShift_Bottom
Lighting.ColorShift_Top = originalLightingProps.ColorShift_Top
Lighting.EnvironmentDiffuseScale = originalLightingProps.EnvironmentDiffuseScale
Lighting.EnvironmentSpecularScale = originalLightingProps.EnvironmentSpecularScale
Lighting.ExposureCompensation = originalLightingProps.ExposureCompensation
Lighting.GeographicLatitude = originalLightingProps.GeographicLatitude
Lighting.GlobalShadows = originalLightingProps.GlobalShadows
Lighting.OutdoorAmbient = originalLightingProps.OutdoorAmbient
end
end)
end
end
local tpWalkHeartbeatConn = nil
local function UpdateTPWalk(enabled)
State.TPWalk = enabled
if tpWalkHeartbeatConn then
tpWalkHeartbeatConn:Disconnect()
tpWalkHeartbeatConn = nil
end
if enabled then
tpWalkHeartbeatConn = RunService.Heartbeat:Connect(function(dt)
if not State.TPWalk then return end
local char = LocalPlayer.Character
if char then
local hrp = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChildOfClass("Humanoid")
if hrp and hum and hum.Health > 0 and hum.MoveDirection.Magnitude > 0 then
local speed = (State.TPWalkSpeed or 3) * 10
hrp.CFrame = hrp.CFrame + (hum.MoveDirection * (speed * dt))
end
end
end)
end
end
local bhopHeartbeatConn = nil
local spaceHeld = false
local mobileJumpHeld = false
local function HookMobileJump()
pcall(function()
local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
local touchGui = playerGui and playerGui:FindFirstChild("TouchGui")
local tcf = touchGui and touchGui:FindFirstChild("TouchControlFrame")
local jumpBtn = tcf and tcf:FindFirstChild("JumpButton")
if jumpBtn then
jumpBtn.InputBegan:Connect(function(inp)
if inp.UserInputType == Enum.UserInputType.Touch or inp.UserInputType == Enum.UserInputType.MouseButton1 then
mobileJumpHeld = true
end
end)
jumpBtn.InputEnded:Connect(function(inp)
if inp.UserInputType == Enum.UserInputType.Touch or inp.UserInputType == Enum.UserInputType.MouseButton1 then
mobileJumpHeld = false
end
end)
end
end)
end
HookMobileJump()
LocalPlayer.CharacterAdded:Connect(function()
task.wait(0.5)
HookMobileJump()
end)
UserInputService.InputBegan:Connect(function(inp, gpe)
if inp.KeyCode == Enum.KeyCode.Space then
spaceHeld = true
elseif inp.UserInputType == Enum.UserInputType.Touch then
local vp = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(800, 600)
if inp.Position.X > vp.X * 0.70 and inp.Position.Y > vp.Y * 0.55 then
mobileJumpHeld = true
end
end
end)
UserInputService.InputEnded:Connect(function(inp, gpe)
if inp.KeyCode == Enum.KeyCode.Space then
spaceHeld = false
elseif inp.UserInputType == Enum.UserInputType.Touch then
mobileJumpHeld = false
end
end)
local function IsJumpHolding()
if spaceHeld then return true end
local isDown = false
pcall(function()
isDown = UserInputService:IsKeyDown(Enum.KeyCode.Space)
end)
if isDown then return true end
if mobileJumpHeld then return true end
return false
end
local function UpdateBhop(enabled)
State.Bhop = enabled
if bhopHeartbeatConn then
bhopHeartbeatCon
