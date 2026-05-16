-- Script Completo: Super Poderes Mobile com Controles de Velocidade
-- Coloque este script em StarterPlayer > StarterCharacterScripts

local player = game.Players:GetPlayerFromCharacter(script.Parent)
local character = script.Parent
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Variáveis de controle
local espEnabled = false
local speedEnabled = false
local jumpEnabled = false
local flyEnabled = false
local flying = false

-- Valores ajustáveis
local speedValue = 50
local jumpValue = 50
local flySpeed = 50

-- Serviços
local userInputService = game:GetService("UserInputService")
local runService = game:GetService("RunService")
local players = game:GetService("Players")
local camera = workspace.CurrentCamera

-- Criar a GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CompleteMobileGui"
screenGui.ResetOnSpawn = false
screenGui.ScreenInsets = Enum.ScreenInsets.None
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Painel principal (lado direito)
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 140, 0, 500)
mainFrame.Position = UDim2.new(1, -150, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0.3
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 15)
corner.Parent = mainFrame

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(1, 0, 0, 35)
titleLabel.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Text = "⚡ PODERES"
titleLabel.BorderSizePixel = 0
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 15)
titleCorner.Parent = titleLabel

-- Função para criar botão
local function createPowerButton(name, icon, yPos, onToggle)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.9, 0, 0, 60)
    button.Position = UDim2.new(0.05, 0, 0, yPos)
    button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextScaled = true
    button.Text = icon .. "\n[OFF]"
    button.BorderSizePixel = 0
    button.Font = Enum.Font.GothamBold
    button.Parent = mainFrame
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 10)
    buttonCorner.Parent = button
    
    local isEnabled = false
    local originalColor = button.BackgroundColor3
    
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    end)
    
    button.MouseLeave:Connect(function()
        if not isEnabled then
            button.BackgroundColor3 = originalColor
        end
    end)
    
    button.MouseButton1Click:Connect(function()
        isEnabled = not isEnabled
        onToggle(isEnabled)
        
        if isEnabled then
            button.Text = icon .. "\n[ON]"
            button.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
        else
            button.Text = icon .. "\n[OFF]"
            button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        end
    end)
    
    return button, function() return isEnabled end
end

-- Botão ESP
local espButton, getEspState = createPowerButton("EspButton", "👁️", 40, function(enabled)
    espEnabled = enabled
end)

-- Botão Super Velocidade
local speedButton, getSpeedState = createPowerButton("SpeedButton", "💨", 110, function(enabled)
    speedEnabled = enabled
end)

-- Botão Super Pulo
local jumpButton, getJumpState = createPowerButton("JumpButton", "🚀", 180, function(enabled)
    jumpEnabled = enabled
end)

-- Botão Fly
local flyButton, getFlyState = createPowerButton("FlyButton", "✈️", 250, function(enabled)
    flyEnabled = enabled
    if not enabled and flying then
        flying = false
        if rootPart:FindFirstChild("BodyVelocity") then
            rootPart:FindFirstChild("BodyVelocity"):Destroy()
        end
    end
end)

-- ==================== PAINEL DE CONTROLE DE VALORES ====================

local controlPanel = Instance.new("Frame")
controlPanel.Name = "ControlPanel"
controlPanel.Size = UDim2.new(0, 140, 0, 200)
controlPanel.Position = UDim2.new(1, -150, 0, 20)
controlPanel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
controlPanel.BorderSizePixel = 0
controlPanel.BackgroundTransparency = 0.3
controlPanel.Parent = screenGui

local controlCorner = Instance.new("UICorner")
controlCorner.CornerRadius = UDim.new(0, 15)
controlCorner.Parent = controlPanel

-- Label de Velocidade
local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(1, 0, 0, 25)
speedLabel.Position = UDim2.new(0, 0, 0, 5)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
speedLabel.TextScaled = true
speedLabel.Text = "VEL: " .. speedValue
speedLabel.Font = Enum.Font.GothamBold
speedLabel.Parent = controlPanel

-- Botões + e - para Velocidade
local speedPlusBtn = Instance.new("TextButton")
speedPlusBtn.Size = UDim2.new(0.45, 0, 0, 25)
speedPlusBtn.Position = UDim2.new(0.05, 0, 0, 30)
speedPlusBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
speedPlusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedPlusBtn.TextScaled = true
speedPlusBtn.Text = "+"
speedPlusBtn.BorderSizePixel = 0
speedPlusBtn.Font = Enum.Font.GothamBold
speedPlusBtn.Parent = controlPanel

local speedPlusCorner = Instance.new("UICorner")
speedPlusCorner.CornerRadius = UDim.new(0, 8)
speedPlusCorner.Parent = speedPlusBtn

local speedMinusBtn = Instance.new("TextButton")
speedMinusBtn.Size = UDim2.new(0.45, 0, 0, 25)
speedMinusBtn.Position = UDim2.new(0.5, 0, 0, 30)
speedMinusBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
speedMinusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedMinusBtn.TextScaled = true
speedMinusBtn.Text = "-"
speedMinusBtn.BorderSizePixel = 0
speedMinusBtn.Font = Enum.Font.GothamBold
speedMinusBtn.Parent = controlPanel

local speedMinusCorner = Instance.new("UICorner")
speedMinusCorner.CornerRadius = UDim.new(0, 8)
speedMinusCorner.Parent = speedMinusBtn

speedPlusBtn.MouseButton1Click:Connect(function()
    speedValue = math.min(speedValue + 10, 200)
    speedLabel.Text = "VEL: " .. speedValue
end)

speedMinusBtn.MouseButton1Click:Connect(function()
    speedValue = math.max(speedValue - 10, 10)
    speedLabel.Text = "VEL: " .. speedValue
end)

-- Label de Pulo
local jumpLabel = Instance.new("TextLabel")
jumpLabel.Name = "JumpLabel"
jumpLabel.Size = UDim2.new(1, 0, 0, 25)
jumpLabel.Position = UDim2.new(0, 0, 0, 65)
jumpLabel.BackgroundTransparency = 1
jumpLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
jumpLabel.TextScaled = true
jumpLabel.Text = "PULO: " .. jumpValue
jumpLabel.Font = Enum.Font.GothamBold
jumpLabel.Parent = controlPanel

-- Botões + e - para Pulo
local jumpPlusBtn = Instance.new("TextButton")
jumpPlusBtn.Size = UDim2.new(0.45, 0, 0, 25)
jumpPlusBtn.Position = UDim2.new(0.05, 0, 0, 90)
jumpPlusBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
jumpPlusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpPlusBtn.TextScaled = true
jumpPlusBtn.Text = "+"
jumpPlusBtn.BorderSizePixel = 0
jumpPlusBtn.Font = Enum.Font.GothamBold
jumpPlusBtn.Parent = controlPanel

local jumpPlusCorner = Instance.new("UICorner")
jumpPlusCorner.CornerRadius = UDim.new(0, 8)
jumpPlusCorner.Parent = jumpPlusBtn

local jumpMinusBtn = Instance.new("TextButton")
jumpMinusBtn.Size = UDim2.new(0.45, 0, 0, 25)
jumpMinusBtn.Position = UDim2.new(0.5, 0, 0, 90)
jumpMinusBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
jumpMinusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpMinusBtn.TextScaled = true
jumpMinusBtn.Text = "-"
jumpMinusBtn.BorderSizePixel = 0
jumpMinusBtn.Font = Enum.Font.GothamBold
jumpMinusBtn.Parent = controlPanel

local jumpMinusCorner = Instance.new("UICorner")
jumpMinusCorner.CornerRadius = UDim.new(0, 8)
jumpMinusCorner.Parent = jumpMinusBtn

jumpPlusBtn.MouseButton1Click:Connect(function()
    jumpValue = math.min(jumpValue + 10, 200)
    jumpLabel.Text = "PULO: " .. jumpValue
end)

jumpMinusBtn.MouseButton1Click:Connect(function()
    jumpValue = math.max(jumpValue - 10, 10)
    jumpLabel.Text = "PULO: " .. jumpValue
end)

-- Label de Fly Speed
local flyLabel = Instance.new("TextLabel")
flyLabel.Name = "FlyLabel"
flyLabel.Size = UDim2.new(1, 0, 0, 25)
flyLabel.Position = UDim2.new(0, 0, 0, 125)
flyLabel.BackgroundTransparency = 1
flyLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
flyLabel.TextScaled = true
flyLabel.Text = "FLY: " .. flySpeed
flyLabel.Font = Enum.Font.GothamBold
flyLabel.Parent = controlPanel

-- Botões + e - para Fly Speed
local flyPlusBtn = Instance.new("TextButton")
flyPlusBtn.Size = UDim2.new(0.45, 0, 0, 25)
flyPlusBtn.Position = UDim2.new(0.05, 0, 0, 150)
flyPlusBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
flyPlusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyPlusBtn.TextScaled = true
flyPlusBtn.Text = "+"
flyPlusBtn.BorderSizePixel = 0
flyPlusBtn.Font = Enum.Font.GothamBold
flyPlusBtn.Parent = controlPanel

local flyPlusCorner = Instance.new("UICorner")
flyPlusCorner.CornerRadius = UDim.new(0, 8)
flyPlusCorner.Parent = flyPlusBtn

local flyMinusBtn = Instance.new("TextButton")
flyMinusBtn.Size = UDim2.new(0.45, 0, 0, 25)
flyMinusBtn.Position = UDim2.new(0.5, 0, 0, 150)
flyMinusBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
flyMinusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyMinusBtn.TextScaled = true
flyMinusBtn.Text = "-"
flyMinusBtn.BorderSizePixel = 0
flyMinusBtn.Font = Enum.Font.GothamBold
flyMinusBtn.Parent = controlPanel

local flyMinusCorner = Instance.new("UICorner")
flyMinusCorner.CornerRadius = UDim.new(0, 8)
flyMinusCorner.Parent = flyMinusBtn

flyPlusBtn.MouseButton1Click:Connect(function()
    flySpeed = math.min(flySpeed + 10, 200)
    flyLabel.Text = "FLY: " .. flySpeed
end)

flyMinusBtn.MouseButton1Click:Connect(function()
    flySpeed = math.max(flySpeed - 10, 10)
    flyLabel.Text = "FLY: " .. flySpeed
end)

-- ==================== SISTEMA ESP ====================

local espBoxes = {}

local function createEspBox(targetPlayer)
    if targetPlayer == player or not targetPlayer.Character then return end
    
    local targetChar = targetPlayer.Character
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    
    if not targetHRP then return end
    
    -- Remover caixa anterior se existir
    if espBoxes[targetPlayer] then
        local oldBox = espBoxes[targetPlayer]
        if oldBox and oldBox.Parent then
            oldBox:Destroy()
        end
    end
    
    -- Criar nova caixa
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "ESPBox_" .. targetPlayer.Name
    billboardGui.Adornee = targetHRP
    billboardGui.MaxDistance = 500
    billboardGui.Size = UDim2.new(4, 0, 6, 0)
    billboardGui.Parent = targetHRP
    
    local frame = Instance.new("Frame")
    frame.BackgroundTransparency = 1
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.Parent = billboardGui
    
    -- Criar outline vermelho
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Thickness = 2
    uiStroke.Color = Color3.fromRGB(255, 0, 0)
    uiStroke.Parent = frame
    
    -- Adicionar nome do jogador
    local nameLabel = Instance.new("TextLabel")
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    nameLabel.TextScaled = true
    nameLabel.Text = targetPlayer.Name
    nameLabel.Size = UDim2.new(1, 0, 0.2, 0)
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = frame
    
    espBoxes[targetPlayer] = billboardGui
end

-- Atualizar ESP
runService.Heartbeat:Connect(function()
    if espEnabled then
        for _, p in pairs(players:GetPlayers()) do
            if p ~= player and p.Character then
                createEspBox(p)
            end
        end
    else
        -- Limpar ESP
        for target, box in pairs(espBoxes) do
            if box and box.Parent then
                box:Destroy()
            end
        end
        espBoxes = {}
    end
end)

-- ==================== SISTEMA DE SUPER VELOCIDADE ====================

runService.RenderStepped:Connect(function()
    if speedEnabled and humanoid.Health > 0 then
        local direction = Vector3.new(0, 0, 0)
        
        if userInputService:IsKeyDown(Enum.KeyCode.W) then
            direction = direction + (rootPart.CFrame.LookVector)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.S) then
            direction = direction - (rootPart.CFrame.LookVector)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.A) then
            direction = direction - (rootPart.CFrame.RightVector)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.D) then
            direction = direction + (rootPart.CFrame.RightVector)
        end
        
        if direction.Magnitude > 0 then
            direction = direction.Unit
            rootPart.Velocity = Vector3.new(direction.X * speedValue, rootPart.Velocity.Y, direction.Z * speedValue)
        end
    end
end)

-- ==================== SISTEMA DE SUPER PULO ====================

userInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Space and jumpEnabled and humanoid.Health > 0 then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        rootPart.AssemblyLinearVelocity = Vector3.new(rootPart.AssemblyLinearVelocity.X, 0, rootPart.AssemblyLinearVelocity.Z)
        rootPart.AssemblyLinearVelocity = rootPart.AssemblyLinearVelocity + Vector3.new(0, jumpValue * 1.5, 0)
    end
end)

-- ==================== SISTEMA DE FLY ====================

userInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.F and flyEnabled then
        flying = not flying
        
        if flying then
            -- Criar BodyVelocity
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Name = "BodyVelocity"
            bodyVelocity.Velocity = Vector3.new(0, 0, 0)
            bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bodyVelocity.Parent = rootPart
        else
            -- Remover BodyVelocity
            if rootPart:FindFirstChild("BodyVelocity") then
                rootPart:FindFirstChild("BodyVelocity"):Destroy()
            end
        end
    end
end)

-- Controle de Fly
runService.RenderStepped:Connect(function()
    if flyEnabled and flying then
        local bodyVelocity = rootPart:FindFirstChild("BodyVelocity")
        if bodyVelocity then
            local direction = Vector3.new(0, 0, 0)
            
            if userInputService:IsKeyDown(Enum.KeyCode.W) then
                direction = direction + (rootPart.CFrame.LookVector)
            end
            if userInputService:IsKeyDown(Enum.KeyCode.S) then
                direction = direction - (rootPart.CFrame.LookVector)
            end
            if userInputService:IsKeyDown(Enum.KeyCode.A) then
                direction = direction - (rootPart.CFrame.RightVector)
            end
            if userInputService:IsKeyDown(Enum.KeyCode.D) then
                direction = direction + (rootPart.CFrame.RightVector)
            end
            if userInputService:IsKeyDown(Enum.KeyCode.Space) then
                direction = direction + Vector3.new(0, 1, 0)
            end
            if userInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
                direction = direction - Vector3.new(0, 1, 0)
            end
            
            if direction.Magnitude > 0 then
                direction = direction.Unit
            end
            
            bodyVelocity.Velocity = direction * flySpeed
        end
    end
end)

-- Limpar quando morrer
humanoid.Died:Connect(function()
    if rootPart:FindFirstChild("BodyVelocity") then
        rootPart:FindFirstChild("BodyVelocity"):Destroy()
    end
    screenGui:Destroy()
end)

print("✅ Script de Super Poderes Carregado!")
