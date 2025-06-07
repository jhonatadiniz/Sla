-- Silent Aim simples para fins educativos/testes internos

-- Serviços necessários
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local RunService = game:GetService("RunService")

-- Configurações
local SilentAimEnabled = true
local AimPart = "Head" -- Alvo: Head ou HumanoidRootPart
local FieldOfView = 150

-- Interface opcional
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local ToggleButton = Instance.new("TextButton", ScreenGui)
ToggleButton.Size = UDim2.new(0, 150, 0, 40)
ToggleButton.Position = UDim2.new(0, 20, 0, 70)
ToggleButton.Text = "Silent Aim ON"
ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
ToggleButton.TextColor3 = Color3.new(1,1,1)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 20

ToggleButton.MouseButton1Click:Connect(function()
	SilentAimEnabled = not SilentAimEnabled
	ToggleButton.Text = SilentAimEnabled and "Silent Aim ON" or "Silent Aim OFF"
end)

-- Função para pegar o inimigo mais próximo dentro do FOV
local function GetClosestTarget()
	local closest = nil
	local shortestDistance = FieldOfView

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team and player.Character and player.Character:FindFirstChild(AimPart) then
			local pos = workspace.CurrentCamera:WorldToScreenPoint(player.Character[AimPart].Position)
			local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude

			if dist < shortestDistance then
				shortestDistance = dist
				closest = player
			end
		end
	end

	return closest
end

-- Manipular função de Raycast (usado por armas)
local mt = getrawmetatable(game)
setreadonly(mt, false)

local oldNamecall = mt.__namecall
mt.__namecall = newcclosure(function(self, ...)
	local args = {...}
	local method = getnamecallmethod()

	if SilentAimEnabled and method == "FindPartOnRayWithIgnoreList" then
		local target = GetClosestTarget()

		if target and target.Character and target.Character:FindFirstChild(AimPart) then
			local origin = workspace.CurrentCamera.CFrame.Position
			local direction = (target.Character[AimPart].Position - origin).Unit * 1000

			args[1] = Ray.new(origin, direction)
			return oldNamecall(self, unpack(args))
		end
	end

	return oldNamecall(self, ...)
end)
