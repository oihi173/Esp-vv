 -- Enemy ESP with Red Highlight + Performance (Low Texture) Option
-- Coloque esse script em StarterGui como LocalScript

local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer

-- Painel UI
local screenGui = Instance.new("ScreenGui", localPlayer.PlayerGui)
screenGui.Name = "ESPPanel"

local espButton = Instance.new("TextButton", screenGui)
espButton.Size = UDim2.new(0, 150, 0, 50)
espButton.Position = UDim2.new(0, 10, 0, 10)
espButton.Text = "Ativar ESP"
espButton.BackgroundColor3 = Color3.fromRGB(200,0,0)

local fosButton = Instance.new("TextButton", screenGui)
fosButton.Size = UDim2.new(0, 150, 0, 50)
fosButton.Position = UDim2.new(0, 10, 0, 70)
fosButton.Text = "Modo Mais Fos"
fosButton.BackgroundColor3 = Color3.fromRGB(100,100,100)

local espActive = false
local fosActive = false

-- Função para adicionar ESP (Highlight vermelho) aos inimigos
function updateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer then
            if player.Team ~= localPlayer.Team then -- Só inimigos
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
            else
                -- Remove highlight do time aliado
                local character = player.Character
                if character and character:FindFirstChild("EnemyESPHighlight") then
                    character.EnemyESPHighlight.Enabled = false
                end
            end
        end
    end
end

-- Função para deixar as texturas mais leves (modo "fos")
function setLowTexture()
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("Texture") or v:IsA("Decal") then
            v.Transparency = fosActive and 0.7 or 0 -- Transparente para menos detalhes
        end
    end
end

-- Botão ESP
espButton.MouseButton1Click:Connect(function()
    espActive = not espActive
    espButton.Text = espActive and "Desativar ESP" or "Ativar ESP"
    updateESP()
end)

-- Botão FOS
fosButton.MouseButton1Click:Connect(function()
    fosActive = not fosActive
    fosButton.Text = fosActive and "Desativar Modo Fos" or "Modo Mais Fos"
    setLowTexture()
end)

-- Atualiza ESP quando players entram/são atualizados
Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        updateESP()
    end)
end)

Players.PlayerRemoving:Connect(function(plr)
    updateESP()
end)

-- Atualiza ESP a cada 2 segundos
while true do
    updateESP()
    wait(2)
end
