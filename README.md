--// LOCAL SCRIPT - StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

---------------------------------------------------
-- GUI
---------------------------------------------------

local gui = Instance.new("ScreenGui")
gui.Parent = player.PlayerGui
gui.ResetOnSpawn = false

local frame = Instance.new("Frame")
frame.Parent = gui
frame.Size = UDim2.new(0,350,0,260)
frame.Position = UDim2.new(0.5,-175,0.5,-130)
frame.BackgroundColor3 = Color3.fromRGB(20,20,25)
frame.BorderSizePixel = 0

local corner = Instance.new("UICorner",frame)
corner.CornerRadius = UDim.new(0,14)

local title = Instance.new("TextLabel")
title.Parent = frame
title.Size = UDim2.new(1,0,0,45)
title.BackgroundTransparency = 1
title.Text = "⚡ ADMIN PANEL"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 24

---------------------------------------------------
-- BUTTON FUNCTION
---------------------------------------------------

local function CreateButton(text,y)

	local button = Instance.new("TextButton")
	button.Parent = frame
	button.Size = UDim2.new(0,300,0,40)
	button.Position = UDim2.new(0.5,-150,0,y)
	button.BackgroundColor3 = Color3.fromRGB(35,35,45)
	button.TextColor3 = Color3.new(1,1,1)
	button.Text = text
	button.Font = Enum.Font.GothamBold
	button.TextSize = 17

	local c = Instance.new("UICorner",button)
	c.CornerRadius = UDim.new(0,10)

	return button
end

---------------------------------------------------
-- BUTTONS
---------------------------------------------------

local espButton = CreateButton("ESP PLAYERS",60)
local aimButton = CreateButton("AIM NPC",115)
local fpsButton = CreateButton("FPS/PING",170)

---------------------------------------------------
-- ESP
---------------------------------------------------

local espEnabled = false

local function AddESP(char)
	if char:FindFirstChild("Highlight") then return end

	local h = Instance.new("Highlight")
	h.FillColor = Color3.fromRGB(255,0,0)
	h.OutlineColor = Color3.fromRGB(255,255,255)
	h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	h.Parent = char
end

espButton.MouseButton1Click:Connect(function()

	espEnabled = not espEnabled

	if espEnabled then
		espButton.Text = "ESP: ON"

		for _,plr in pairs(Players:GetPlayers()) do
			if plr ~= player and plr.Character then
				AddESP(plr.Character)
			end
		end

	else
		espButton.Text = "ESP: OFF"

		for _,plr in pairs(Players:GetPlayers()) do
			if plr.Character and plr.Character:FindFirstChild("Highlight") then
				plr.Character.Highlight:Destroy()
			end
		end
	end
end)

---------------------------------------------------
-- AIM NPC
---------------------------------------------------

local aimEnabled = false

local function GetClosestDummy()

	local closest
	local dist = math.huge

	for _,obj in pairs(workspace:GetChildren()) do

		if obj.Name == "Dummy" and obj:FindFirstChild("Head") then

			local pos,visible =
				camera:WorldToViewportPoint(obj.Head.Position)

			if visible then

				local magnitude = (
					Vector2.new(pos.X,pos.Y)
					-
					Vector2.new(
						camera.ViewportSize.X/2,
						camera.ViewportSize.Y/2
					)
				).Magnitude

				if magnitude < dist then
					dist = magnitude
					closest = obj.Head
				end
			end
		end
	end

	return closest
end

aimButton.MouseButton1Click:Connect(function()

	aimEnabled = not aimEnabled

	if aimEnabled then
		aimButton.Text = "AIM NPC: ON"
	else
		aimButton.Text = "AIM NPC: OFF"
	end
end)

RunService.RenderStepped:Connect(function()

	if
