
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

-- Jogador local e câmera
local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- Variáveis de controle
local espEnabled = false
local aimbotEnabled = false
local teamCheck = false
local wallCheck = false
local glowEnabled = false     -- Novo: controle para o efeito glow
local targetPart = "Head"
local fovRadius = 150
local showFov = false
local fovRainbow = false
local espObjects = {}
local glowObjects = {}        -- Novo: objetos para o efeito glow
local hue = 0
local smoothingFactor = 0.5   -- Novo: suavização do aimbot (0-1)

-- Cores e estilos
local COLORS = {
    PRIMARY = Color3.fromRGB(25, 25, 25),
    SECONDARY = Color3.fromRGB(35, 35, 35),
    ACCENT = Color3.fromRGB(255, 200, 0),
    ENABLED = Color3.fromRGB(0, 200, 0),
    DISABLED = Color3.fromRGB(200, 0, 0),
    TEXT = Color3.fromRGB(255, 255, 255)
}

-- UI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ESP_UI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Proteção contra detecção
pcall(function()
    if syn then
        syn.protect_gui(ScreenGui)
        ScreenGui.Parent = game.CoreGui
    else
        ScreenGui.Parent = game.CoreGui
    end
end)

-- Frame principal com sombra e arredondamento
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 250, 0, 430)
MainFrame.Position = UDim2.new(0, 20, 0, 100)
MainFrame.BackgroundColor3 = COLORS.PRIMARY
MainFrame.BorderSizePixel = 0
MainFrame.BackgroundTransparency = 0.1
MainFrame.Active = true
MainFrame.Draggable = true

-- Efeito de sombra
local Shadow = Instance.new("ImageLabel", MainFrame)
Shadow.Size = UDim2.new(1, 30, 1, 30)
Shadow.Position = UDim2.new(0, -15, 0, -15)
Shadow.BackgroundTransparency = 1
Shadow.Image = "rbxassetid://5028857084"
Shadow.ImageColor3 = Color3.new(0, 0, 0)
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(24, 24, 276, 276)
Shadow.ImageTransparency = 0.5
Shadow.ZIndex = 0

-- Barra de título
local TitleBar = Instance.new("Frame", MainFrame)
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = COLORS.ACCENT
TitleBar.BorderSizePixel = 0

-- Título
local Title = Instance.new("TextLabel", TitleBar)
Title.Text = "💸 HUB POBREZA! 💸"
Title.Size = UDim2.new(1, -10, 1, 0)
Title.Position = UDim2.new(0, 5, 0, 0)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(0, 0, 0)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Área de conteúdo
local ContentFrame = Instance.new("Frame", MainFrame)
ContentFrame.Size = UDim2.new(1, -20, 1, -50)
ContentFrame.Position = UDim2.new(0, 10, 0, 45)
ContentFrame.BackgroundTransparency = 1
ContentFrame.BorderSizePixel = 0

local UIListLayout = Instance.new("UIListLayout", ContentFrame)
UIListLayout.Padding = UDim.new(0, 8)
UIListLayout.FillDirection = Enum.FillDirection.Vertical
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayout.VerticalAlignment = Enum.VerticalAlignment.Top

-- Função para criar seções
local function createSection(name)
    local section = Instance.new("Frame", ContentFrame)
    section.Size = UDim2.new(1, 0, 0, 25)
    section.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", section)
    label.Text = name
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = COLORS.ACCENT
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    return section
end

-- Função botão com estado e efeito hover
local function createToggleButton(name, getState, onToggle)
    local button = Instance.new("Frame", ContentFrame)
    button.Size = UDim2.new(1, 0, 0, 35)
    button.BackgroundColor3 = COLORS.SECONDARY
    button.BorderSizePixel = 0
    
    local UICorner = Instance.new("UICorner", button)
    UICorner.CornerRadius = UDim.new(0, 6)
    
    local statusIndicator = Instance.new("Frame", button)
    statusIndicator.Size = UDim2.new(0, 16, 0, 16)
    statusIndicator.Position = UDim2.new(0, 10, 0.5, -8)
    statusIndicator.BorderSizePixel = 0
    
    local statusCorner = Instance.new("UICorner", statusIndicator)
    statusCorner.CornerRadius = UDim.new(0, 8)
    
    local label = Instance.new("TextLabel", button)
    label.Text = name
    label.Size = UDim2.new(1, -40, 1, 0)
    label.Position = UDim2.new(0, 35, 0, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = COLORS.TEXT
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local hitBox = Instance.new("TextButton", button)
    hitBox.Size = UDim2.new(1, 0, 1, 0)
    hitBox.BackgroundTransparency = 1
    hitBox.Text = ""
    
    local function updateColor()
        local enabled = getState()
        statusIndicator.BackgroundColor3 = enabled and COLORS.ENABLED or COLORS.DISABLED
    end
    
    hitBox.MouseButton1Click:Connect(function()
        onToggle()
        updateColor()
    end)
    
    -- Efeitos hover
    hitBox.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    end)
    
    hitBox.MouseLeave:Connect(function()
        button.BackgroundColor3 = COLORS.SECONDARY
    end)
    
    updateColor()
    return button
end

-- Função para slider
local function createSlider(name, min, max, default, onChanged)
    local sliderContainer = Instance.new("Frame", ContentFrame)
    sliderContainer.Size = UDim2.new(1, 0, 0, 50)
    sliderContainer.BackgroundColor3 = COLORS.SECONDARY
    sliderContainer.BorderSizePixel = 0
    
    local UICorner = Instance.new("UICorner", sliderContainer)
    UICorner.CornerRadius = UDim.new(0, 6)
    
    local label = Instance.new("TextLabel", sliderContainer)
    label.Text = name .. ": " .. default
    label.Size = UDim2.new(1, -20, 0, 20)
    label.Position = UDim2.new(0, 10, 0, 5)
    label.BackgroundTransparency = 1
    label.TextColor3 = COLORS.TEXT
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local sliderBg = Instance.new("Frame", sliderContainer)
    sliderBg.Size = UDim2.new(1, -20, 0, 6)
    sliderBg.Position = UDim2.new(0, 10, 0, 30)
    sliderBg.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    sliderBg.BorderSizePixel = 0
    
    local sliderBgCorner = Instance.new("UICorner", sliderBg)
    sliderBgCorner.CornerRadius = UDim.new(0, 3)
    
    local sliderFill = Instance.new("Frame", sliderBg)
    sliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    sliderFill.BackgroundColor3 = COLORS.ACCENT
    sliderFill.BorderSizePixel = 0
    
    local sliderFillCorner = Instance.new("UICorner", sliderFill)
    sliderFillCorner.CornerRadius = UDim.new(0, 3)
    
    local sliderButton = Instance.new("TextButton", sliderBg)
    sliderButton.Size = UDim2.new(1, 0, 1, 0)
    sliderButton.BackgroundTransparency = 1
    sliderButton.Text = ""
    
    local currentValue = default
    
    local function updateSlider(input)
        local pos = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
        local value = math.floor(min + (max - min) * pos)
        currentValue = value
        sliderFill.Size = UDim2.new(pos, 0, 1, 0)
        label.Text = name .. ": " .. value
        onChanged(value)
    end
    
    sliderButton.MouseButton1Down:Connect(function()
        local input = UserInputService.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement then
                updateSlider(input)
            end
        end)
        
        UserInputService.InputEnded:Connect(function(inputObject)
            if inputObject.UserInputType == Enum.UserInputType.MouseButton1 then
                input:Disconnect()
            end
        end)
    end)
    
    return sliderContainer, function() return currentValue end
end

-- Função seletor
local function createSelector(name, options, default, onChanged)
    local selector = Instance.new("Frame", ContentFrame)
    selector.Size = UDim2.new(1, 0, 0, 35)
    selector.BackgroundColor3 = COLORS.SECONDARY
    selector.BorderSizePixel = 0
    
    local UICorner = Instance.new("UICorner", selector)
    UICorner.CornerRadius = UDim.new(0, 6)
    
    local label = Instance.new("TextLabel", selector)
    label.Text = name .. ": " .. options[default]
    label.Size = UDim2.new(1, -80, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = COLORS.TEXT
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local nextButton = Instance.new("TextButton", selector)
    nextButton.Size = UDim2.new(0, 60, 0, 25)
    nextButton.Position = UDim2.new(1, -70, 0.5, -12.5)
    nextButton.BackgroundColor3 = COLORS.ACCENT
    nextButton.BorderSizePixel = 0
    nextButton.Text = "Mudar"
    nextButton.TextColor3 = Color3.new(0, 0, 0)
    nextButton.Font = Enum.Font.GothamSemibold
    nextButton.TextSize = 14
    
    local nextButtonCorner = Instance.new("UICorner", nextButton)
    nextButtonCorner.CornerRadius = UDim.new(0, 4)
    
    local currentIndex = default
    
    nextButton.MouseButton1Click:Connect(function()
        currentIndex = currentIndex % #options + 1
        label.Text = name .. ": " .. options[currentIndex]
        onChanged(options[currentIndex])
    end)
    
    -- Efeitos hover
    nextButton.MouseEnter:Connect(function()
        nextButton.BackgroundColor3 = Color3.fromRGB(255, 220, 50)
    end)
    
    nextButton.MouseLeave:Connect(function()
        nextButton.BackgroundColor3 = COLORS.ACCENT
    end)
    
    return selector
end

-- Seções da UI
createSection("VISUALIZAÇÃO")

-- Botões de ESP
createToggleButton("ESP", function() return espEnabled end, function() espEnabled = not espEnabled end)
createToggleButton("ESP Glow", function() return glowEnabled end, function() glowEnabled = not glowEnabled end)
createToggleButton("Verificar Time", function() return teamCheck end, function() teamCheck = not teamCheck end)
createToggleButton("Verificar Parede", function() return wallCheck end, function() wallCheck = not wallCheck end)

createSection("AIMBOT")

-- Botões de Aimbot
createToggleButton("Aimbot", function() return aimbotEnabled end, function() aimbotEnabled = not aimbotEnabled end)
createToggleButton("Exibir FOV", function() return showFov end, function() showFov = not showFov end)
createToggleButton("FOV RGB", function() return fovRainbow end, function() fovRainbow = not fovRainbow end)

-- Slider para FOV
local fovSlider, getFov = createSlider("FOV", 50, 500, fovRadius, function(value)
    fovRadius = value
    if fovCircle then
        fovCircle.Radius = value
    end
end)

-- Slider para suavização do aimbot
local smoothSlider = createSlider("Suavização Aimbot", 0, 10, smoothingFactor * 10, function(value)
    smoothingFactor = value / 10
end)

-- Seletor de partes do corpo
createSelector("Parte Alvo", {"Head", "HumanoidRootPart", "Torso"}, 1, function(value)
    targetPart = value
end)

-- Efeito Glow
local function updateGlowForPlayer(player)
    if not glowEnabled or not espEnabled then
        if glowObjects[player] then
            for _, highlighter in pairs(glowObjects[player]) do
                highlighter:Destroy()
            end
            glowObjects[player] = nil
        end
        return
    end

    if player == LocalPlayer then return end
    if teamCheck and player.Team == LocalPlayer.Team then return end
    
    local character = player.Character
    if not character then return end
    
    if not glowObjects[player] then
        glowObjects[player] = {}
    end
    
    -- Limpa highlighters antigos
    for _, highlighter in pairs(glowObjects[player]) do
        highlighter:Destroy()
    end
    glowObjects[player] = {}
    
    -- Adiciona novos highlighters
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            -- Determina a cor do glow baseado no time
            local glowColor = player.Team == LocalPlayer.Team and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
            
            -- Cria o highlighter
            local highlighter = Instance.new("Highlight")
            highlighter.Name = "ESPHighlight"
            highlighter.FillColor = glowColor
            highlighter.OutlineColor = glowColor
            highlighter.FillTransparency = 0.7
            highlighter.OutlineTransparency = 0
            highlighter.Adornee = part
            highlighter.Parent = game.CoreGui
            
            table.insert(glowObjects[player], highlighter)
        end
    end
end

-- FOV Círculo
local fovCircle = Drawing.new("Circle")
fovCircle.Visible = showFov
fovCircle.Thickness = 2
fovCircle.Radius = fovRadius
fovCircle.Transparency = 1
fovCircle.Color = Color3.new(1, 1, 1)
fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

-- ESP
local function createESP(player)
    if player == LocalPlayer or espObjects[player] then return end
    
    -- Box do ESP
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.new(1, 1, 1)
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false
    
    -- Nome do jogador
    local nameTag = Drawing.new("Text")
    nameTag.Visible = false
    nameTag.Center = true
    nameTag.Outline = true
    nameTag.Font = 2
    nameTag.Size = 14
    nameTag.Color = Color3.new(1, 1, 1)
    
    -- Distância
    local distanceText = Drawing.new("Text")
    distanceText.Visible = false
    distanceText.Center = true
    distanceText.Outline = true
    distanceText.Font = 2
    distanceText.Size = 12
    distanceText.Color = Color3.new(1, 1, 1)
    
    espObjects[player] = {
        box = box,
        nameTag = nameTag,
        distanceText = distanceText
    }
end

local function removeESP(player)
    if espObjects[player] then
        for _, drawing in pairs(espObjects[player]) do
            drawing:Remove()
        end
        espObjects[player] = nil
    end
    
    if glowObjects[player] then
        for _, highlighter in pairs(glowObjects[player]) do
            highlighter:Destroy()
        end
        glowObjects[player] = nil
    end
end

-- Setup inicial
for _, player in ipairs(Players:GetPlayers()) do
    createESP(player)
end

Players.PlayerAdded:Connect(createESP)
Players.PlayerRemoving:Connect(removeESP)

-- Verificação de parede
local function isVisible(part)
    if not wallCheck then return true end
    local origin = Camera.CFrame.Position
    local direction = (part.Position - origin)
    local ray = Ray.new(origin, direction.Unit * math.min(direction.Magnitude, 1000))
    local hitPart, hitPosition = Workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character}, false, true)
    if hitPart then
        return hitPart:IsDescendantOf(part.Parent)
    end
    return false
end

-- Aimbot alvo
local function getClosestPart()
    local closestPart = nil
    local shortestDist = fovRadius
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(targetPart) then
            if teamCheck and player.Team == LocalPlayer.Team then continue end
            local part = player.Character[targetPart]
            local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
            local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
            if onScreen and dist < shortestDist and isVisible(part) then
                shortestDist = dist
                closestPart = part
            end
        end
    end
    return closestPart
end

-- Suavização do Aimbot
local function smoothLookAt(target)
    local startCFrame = Camera.CFrame
    local endCFrame = CFrame.new(Camera.CFrame.Position, target.Position)
    local newCFrame = startCFrame:Lerp(endCFrame, 1 - smoothingFactor)
    Camera.CFrame = newCFrame
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    -- Atualiza FOV
    fovCircle.Visible = showFov and aimbotEnabled
    fovCircle.Radius = fovRadius
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    
    -- Efeito de arco-íris para o FOV
    if fovRainbow then
        hue = (hue + 0.005) % 1
        fovCircle.Color = Color3.fromHSV(hue, 1, 1)
    else
        fovCircle.Color = COLORS.ACCENT
    end

    -- Atualiza ESP e Glow
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            -- Update ESP
            local esp = espObjects[player]
            if not esp then
                createESP(player)
                esp = espObjects[player]
            end
            
            local char = player.Character
            if char and char:FindFirstChild("HumanoidRootPart") and espEnabled then
                if teamCheck and player.Team == LocalPlayer.Team then
                    esp.nameTag.Visible = false
                    esp.box.Visible = false
                    esp.distanceText.Visible = false
                else
                    local humanoid = char:FindFirstChildOfClass("Humanoid")
                    local hrp = char.HumanoidRootPart
                    local head = char:FindFirstChild("Head")
                    
                    if hrp and head and humanoid then
                        -- Cálculo da caixa do ESP
                        local rootPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                        
                        if onScreen and isVisible(hrp) then
                            -- Cálculo de distância
                            local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
                            local displayDistance = math.floor(distance)
                            
                            -- Atualizando nome
                            esp.nameTag.Position = Vector2.new(rootPos.X, rootPos.Y - 40)
                            esp.nameTag.Text = player.Name
                            esp.nameTag.Visible = true
                            
                            -- Atualizando distância
                            esp.distanceText.Position = Vector2.new(rootPos.X, rootPos.Y + 20)
                            esp.distanceText.Text = displayDistance .. "m"
                            esp.distanceText.Visible = true
                            
                            -- Determina cor baseada na saúde
                            local healthPercent = humanoid.Health / humanoid.MaxHealth
                            local healthColor = Color3.fromRGB(
                                255 * (1 - healthPercent),
                                255 * healthPercent,
                                0
                            )
                            
                            esp.nameTag.Color = healthColor
                            esp.box.Color = healthColor
                            
                            -- Atualizando box
                            local size = Vector3.new(4, 6, 0) * (1 / (rootPos.Z * 0.001 + 1))
                            esp.box.Size = Vector2.new(size.X, size.Y)
                            esp.box.Position = Vector2.new(rootPos.X - size.X/2, rootPos.Y - size.Y/2)
                            esp.box.Visible = true
                        else
