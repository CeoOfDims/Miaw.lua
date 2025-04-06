local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local player = Players.LocalPlayer
local camera = Workspace.CurrentCamera

-- === SETTINGS ===
local TARGET_NAME = "Rake" -- Ganti nama ini kalau nama musuh bukan "Rake"
local NAME_TAG = "The Rake"
local WALK_SPEED = 35
local AIMBOT_RANGE = 30

-- === FULLBRIGHT ===
Lighting.Ambient = Color3.new(1, 1, 1)
Lighting.OutdoorAmbient = Color3.new(1, 1, 1)
Lighting.Brightness = 5
Lighting.ClockTime = 12
Lighting.FogEnd = 1000000

-- === ANTI LAG ===
for _, v in pairs(Workspace:GetDescendants()) do
	if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
		v.Enabled = false
	elseif v:IsA("Decal") then
		v.Transparency = 1
	end
end

local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

-- Blacklist keyword
local blacklistNames = {
	"tree", "bush", "plant", "leaf", "grass", "flower", "mushroom",
	"fence", "locker", "metallantern", "rakesbed", "jsmodel", "leavelesspine",
	"pole", "sign", "oak", "roofbarier", "trash", "shed", "collision",
	"boards", "tent", "campfire", "fountine", "lantern", "doorflameshop",
	"ceilinglight", "details", "map", "windowflames", "baretta", "board",
	"leverbox", "blackwindow", "firetower", "opper", "tited", "well",
	"credit_tooldrop", "bat_tooldrop", "adrenaline_tooldrop", "polaroid_tooldrop",
	"firstaidkit_tooldrop", "metalpipe_tooldrop", "barn owl", "earth globe",
	"bottle", "taxidermy", "backwindow", "wolf skull", "windowframes",
	"mounted_fish", "doorframe", "rope", "roomlight", "window", "crate",
	"lightswitch", "door", "bathtub", "toilet_low", "cabinet", "cornercabinet",
	"smartwatch_tooldrop", "flashlight_tooldrop", "flaregun_toolsdrop", -- typo amanin
	"flaregun_tooldrop", "crowbar_tooldrop", "fountain", "antiquehelmet_tooldrop",
	"cabin inside", "cabin outside", "mounted officer", "deer_trophy", "hallwaylight", "melakwa",
	"matthew", "room1light", "campsite", "cavecampsite", "interactions",
	"wire", "cabinv2", "cabinvines", "cabin", "walls", "hillcampsite",
	"model", "shop", "rake", "taxidermyfish_stand",
	"park entrance", "dead-end", "restricted area", "mineshaft",
	"tower-1 upper", "tower-1 lower", "tower-2", "radio_tooldrop",
	"emfreader_tooldrop" -- tambahan terbaru
}

local tagged = {}

local function isBlacklisted(name)
	name = name:lower()
	for _, word in pairs(blacklistNames) do
		if name:find(word) then
			return true
		end
	end
	return false
end

local function createESP(part, labelName)
	if part:FindFirstChild("BuildingESP") then return end
	local billboard = Instance.new("BillboardGui")
	billboard.Name = "BuildingESP"
	billboard.Adornee = part
	billboard.Size = UDim2.new(0, 100, 0, 20)
	billboard.StudsOffset = Vector3.new(0, 5, 0)
	billboard.AlwaysOnTop = true

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.Text = labelName
	label.TextColor3 = Color3.fromRGB(255, 215, 0)
	label.TextStrokeTransparency = 0.5
	label.Font = Enum.Font.SourceSansBold
	label.TextScaled = true
	label.Parent = billboard

	billboard.Parent = part
end

local function scan()
	for _, obj in pairs(Workspace:GetDescendants()) do
		if not tagged[obj] and obj:IsA("Model") and not isBlacklisted(obj.Name) then
			if not obj:FindFirstChildOfClass("Humanoid") then
				local primary = obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart")
				if primary then
					createESP(primary, obj.Name)
					tagged[obj] = true
				end
			end
		end
	end
end

RunService.RenderStepped:Connect(function()
	pcall(scan)
end)

-- === WALKSPEED ===
local function applyWalkSpeed()
	local char = player.Character or player.CharacterAdded:Wait()
	local humanoid = char:WaitForChild("Humanoid", 5)
	if humanoid then
		humanoid.WalkSpeed = WALK_SPEED
	end
end

player.CharacterAdded:Connect(function()
	task.wait(0.5)
	applyWalkSpeed()
end)

if player.Character then
	applyWalkSpeed()
end

-- === UI: MADE BY CeoOfDims ===
local function createCreditUI()
	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "CeoOfDimsUI"
	screenGui.ResetOnSpawn = false
	screenGui.Parent = player:WaitForChild("PlayerGui")

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0, 200, 0, 30)
	label.Position = UDim2.new(0, 10, 1, -40)
	label.BackgroundTransparency = 1
	label.Text = "Made by CeoOfDims"
	label.TextColor3 = Color3.fromRGB(255, 255, 255)
	label.TextStrokeTransparency = 0.5
	label.Font = Enum.Font.GothamBold
	label.TextSize = 16
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = screenGui
end

createCreditUI()

-- === ESP & AIMBOT ===
local function getTarget()
	for _, model in pairs(Workspace:GetChildren()) do
		if model:IsA("Model") and model.Name:lower():find(TARGET_NAME:lower()) then
			local root = model:FindFirstChild("HumanoidRootPart") or model:FindFirstChildWhichIsA("BasePart")
			if root then
				return model, root
			end
		end
	end
	return nil, nil
end

local function createESP(target, adornee)
	if target:FindFirstChild("ESP") then return end

	local esp = Instance.new("BillboardGui")
	esp.Name = "ESP"
	esp.Adornee = adornee
	esp.AlwaysOnTop = true
	esp.Size = UDim2.new(0, 100, 0, 40)
	esp.StudsOffset = Vector3.new(0, 3, 0)

	local text = Instance.new("TextLabel")
	text.Size = UDim2.new(1, 0, 1, 0)
	text.BackgroundTransparency = 1
	text.TextColor3 = Color3.fromRGB(255, 0, 0)
	text.TextStrokeTransparency = 0
	text.TextScaled = true
	text.Font = Enum.Font.GothamBold
	text.Text = NAME_TAG
	text.Parent = esp

	esp.Parent = target
end

RunService.RenderStepped:Connect(function()
	local char = player.Character
	if not char then return end
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then return end

	local targetModel, targetPart = getTarget()
	if targetModel and targetPart then
		createESP(targetModel, targetPart)

		local dist = (targetPart.Position - root.Position).Magnitude

		local esp = targetModel:FindFirstChild("ESP")
		if esp and esp:FindFirstChildOfClass("TextLabel") then
			esp.TextLabel.Text = string.format("%s (%.1f)", NAME_TAG, dist)
		end

		if dist <= AIMBOT_RANGE then
			camera.CFrame = CFrame.new(camera.CFrame.Position, targetPart.Position)
		end
	end
end)
