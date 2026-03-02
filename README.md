local ReplicatedStorage = game:GetService("ReplicatedStorage")
local remote = ReplicatedStorage:WaitForChild("AdminControl"):WaitForChild("ToggleAntiLag")

local player = game.Players.LocalPlayer

local gui = Instance.new("ScreenGui")
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 150, 0, 80)
frame.Position = UDim2.new(0, 20, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, -10, 0, 40)
button.Position = UDim2.new(0, 5, 0, 20)
button.Text = "AntiLag: ON"
button.BackgroundColor3 = Color3.fromRGB(0,170,0)
button.TextColor3 = Color3.new(1,1,1)

local enabled = true

button.MouseButton1Click:Connect(function()
	enabled = not enabled
	
	if enabled then
		button.Text = "AntiLag: ON"
		button.BackgroundColor3 = Color3.fromRGB(0,170,0)
	else
		button.Text = "AntiLag: OFF"
		button.BackgroundColor3 = Color3.fromRGB(170,0,0)
	end
	
	remote:FireServer(enabled)
end)
