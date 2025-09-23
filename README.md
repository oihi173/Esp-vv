--[[ 
ESP com destaque vermelho em todos jogadores + modo performance (textura fosca)
- Coloque como LocalScript em StarterGui.
- Painel com fundo, botão hambúrguer para mostrar/ocultar, e arrastável horizontalmente.
]]

local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer

-- Painel UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESPPanel"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer.PlayerGui

local panelFrame = Instance.new("Frame", screenGui)
panelFrame.Size = UDim2.new(0, 170, 0, 140)
panelFrame.Position = UDim2.new(0, 10, 0, 10)
panelFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
panelFrame.BorderSizePixel = 2
panelFrame.BorderColor3 = Color3.fromRGB(200,0,0)
panelFrame.Active = true
panelFrame.Draggable = false -- Usaremos drag customizado

local titleLabel = Instance.new("TextLabel", panelFrame)
titleLabel.Size = UDim2.new(1, 0, 0, 30)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.Text = "Painel ESP"
titleLabel.BackgroundTransparency = 1
titleLabel.TextColor3 = Color3.new(1,1,1)
titleLabel.Font = Enum.Font.SourceSansBold
titleLabel.TextSize = 20

local hamburgerBtn = Instance.new("TextButton", panelFrame)
hamburgerBtn.Size = UDim2.new(0, 30, 0, 30)
hamburgerBtn.Position = UDim2.new(1, -35, 0, 0)
hamburgerBtn.Text = "≡"
hamburgerBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
hamburgerBtn.TextColor3 = Color3.new(1,1,1)
hamburgerBtn.BorderSizePixel = 0

local contentFrame = Instance.new("Frame", panelFrame)
contentFrame.Size = UDim2.new(1, -10, 1, -40)
contentFrame.Position = UDim2.new(0, 5, 0, 35)
contentFrame.BackgroundTransparency = 1

local espButton = Instance.new("TextButton", contentFrame)
espButton.Size = UDim2.new(1, 0, 0, 40)
espButton.Position = UDim2.new(0, 0, 0, 0)
espButton.Text = "Ativar ESP"
espButton.BackgroundColor3 = Color3.fromRGB(200,0,0)
espButton.TextColor3 = Color3.new(1,1,1)
espButton.Font = Enum.Font.SourceSansBold
espButton.TextSize = 18

local fosButton = Instance.new("TextButton", contentFrame)
fosButton.Size = UDim2.new(1, 0, 0, 40)
fosButton.Position = UDim2.new(0, 0, 0, 50)
fosButton.Text = "Modo Mais Fos"
fosButton.BackgroundColor3 = Color3.fromRGB(100,100,100)
fosButton.TextColor3 = Color3.new(1,1,1)
fosButton.Font = Enum.Font.SourceSansBold
fosButton.TextSize = 18

-- Mostrar/esconder painel
local panelVisible = true
hamburgerBtn.MouseButton1Click:Connect(function()
    panelVisible = not panelVisible
    contentFrame.Visible = panelVisible
    espButton.Visible = panelVisible
    fosButton.Visible = panelVisible
end)

-- Arrastar painel horizontalmente pelo titleLabel
local dragging = false
local dragStart = nil
local startPos = nil

titleLabel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = panelFrame.Position
    end
end)
titleLabel.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        panelFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset)
    end
end)

local espActive = false
local fosActive = false

-- Função ESP: pega TODOS jogadores
function updateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer then
            local character = player.Character
            if character then
                if not character:FindFirstChild("EnemyESPHighlight") then
                    local highlight = Instance.new("Highlight", character)
                    highlight.Name = "EnemyESPHighlight"
                    highlight.FillColor = Color3.fromRGB(255,0,0)
                    highlight.OutlineColor = Color3.fromRGB(255,0,0)
                    highlight.Enabled = espActive
                else
                    character.EnemyESPHighlight.Enabled = espActive
                end
            end
        end
    end
end

-- Função para modo "fosco" (baixo textura)
function setLowTexture()
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("Texture") or v:IsA("Decal") then
            v.Transparency = fosActive and 0.7 or 0
        end
    end
end

espButton.MouseButton1Click:Connect(function()
    espActive = not espActive
    espButton.Text = espActive and "Desativar ESP" or "Ativar ESP"
    updateESP()
end)

fosButton.MouseButton1Click:Connect(function()
    fosActive = not fosActive
    fosButton.Text = fosActive and "Desativar Modo Fos" or "Modo Mais Fos"
    setLowTexture()
end)

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        updateESP()
    end)
end)
Players.PlayerRemoving:Connect(function(plr)
    updateESP()
end)

-- Atualiza ESP periodicamente
while true do
    updateESP()
    wait(2)
end
