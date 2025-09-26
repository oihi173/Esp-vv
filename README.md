local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "TeleportGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = playerGui

local tweenInfo = TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

-- Lista de teleportes
local teleports = {
    {name = "Local 1", pos = Vector3.new(4009.78, 21.37, -6705.96)},
    {name = "Local 2", pos = Vector3.new(4125.84, 37.00, -6733.84)},
    {name = "Local 3", pos = Vector3.new(4124.55, 21.37, -6744.80)},
    {name = "Local 4", pos = Vector3.new(3953.47, 21.00, -6689.76)}
}

-- Painel principal (base)
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,260,0,60)
frame.Position = UDim2.new(0.5,-130,0.7,0)
frame.AnchorPoint = Vector2.new(0.5,0)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BackgroundTransparency = 0.15
frame.BorderSizePixel = 0
frame.ClipsDescendants = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,12)

-- Botão abrir/fechar
local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0,36,0,36)
openBtn.Position = UDim2.new(1,-42,0,12)
openBtn.AnchorPoint = Vector2.new(1,0)
openBtn.Text = "+"
openBtn.Font = Enum.Font.SourceSansBold
openBtn.TextScaled = true
openBtn.BackgroundTransparency = 0.1
openBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
openBtn.TextColor3 = Color3.new(1,1,1)
openBtn.Parent = frame
Instance.new("UICorner", openBtn).CornerRadius = UDim.new(0,8)

-- ScrollingFrame
local listFrame = Instance.new("ScrollingFrame")
listFrame.Size = UDim2.new(1,0,0,250)
listFrame.Position = UDim2.new(0,0,0,60)
listFrame.BackgroundTransparency = 1
listFrame.BorderSizePixel = 0
listFrame.ScrollBarThickness = 6
listFrame.Visible = false
listFrame.Parent = frame
listFrame.CanvasSize = UDim2.new(0,0,0,#teleports*46 + 50)
local layout = Instance.new("UIListLayout", listFrame)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0,6)

-- Função Teleporte (suporta cadeira)
local function teleportTo(pos)
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid.SeatPart then
        local seat = humanoid.SeatPart
        TweenService:Create(seat, tweenInfo, {CFrame = CFrame.new(pos + Vector3.new(0,3,0))}):Play()
    else
        TweenService:Create(hrp, tweenInfo, {CFrame = CFrame.new(pos + Vector3.new(0,3,0))}):Play()
    end
end

-- Auto Teleport
local autoTeleporting = false
local autoDelay = 0.5 -- meio segundo entre cada clique

local autoBtn = Instance.new("TextButton")
autoBtn.Size = UDim2.new(1,-12,0,40)
autoBtn.Position = UDim2.new(0,6,0,(#teleports*46)+10)
autoBtn.BackgroundTransparency = 0.08
autoBtn.BackgroundColor3 = Color3.fromRGB(80,50,50)
autoBtn.BorderSizePixel = 0
autoBtn.Text = "Auto Teleport: OFF"
autoBtn.Font = Enum.Font.SourceSansBold
autoBtn.TextSize = 18
autoBtn.TextColor3 = Color3.new(1,1,1)
autoBtn.Parent = listFrame
Instance.new("UICorner", autoBtn).CornerRadius = UDim.new(0,8)

autoBtn.MouseButton1Click:Connect(function()
    autoTeleporting = not autoTeleporting
    autoBtn.Text = "Auto Teleport: "..(autoTeleporting and "ON" or "OFF")
    if autoTeleporting then
        task.spawn(function()
            while autoTeleporting do
                for _, info in ipairs(teleports) do
                    if not autoTeleporting then break end
                    for i=1,3 do -- 3 cliques antes de ir pro próximo
                        if not autoTeleporting then break end
                        teleportTo(info.pos)
                        task.wait(autoDelay)
                    end
                end
            end
        end)
    end
end)

-- Abrir/fechar painel
local open = false
openBtn.MouseButton1Click:Connect(function()
    open = not open
    if open then
        TweenService:Create(frame, TweenInfo.new(0.25), {Size=UDim2.new(0,260,0,360)}):Play()
        listFrame.Visible = true
        openBtn.Text = "×"
    else
        TweenService:Create(frame, TweenInfo.new(0.25), {Size=UDim2.new(0,260,0,60)}):Play()
        task.delay(0.26,function() listFrame.Visible = false end)
        openBtn.Text = "+"
    end
end)

-- Drag
local dragging, dragStart, startPos
local function updateDrag(input)
    if dragging then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(
            0, math.clamp(startPos.X.Offset + delta.X,0,gui.AbsoluteSize.X - frame.Size.X.Offset),
            0, math.clamp(startPos.Y.Offset + delta.Y,0,gui.AbsoluteSize.Y - frame.Size.Y.Offset)
        )
    end
end
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End or input.UserInputState == Enum.UserInputState.Cancel then
                dragging = false
            end
        end)
    end
end)
frame.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
        updateDrag(input)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
        updateDrag(input)
    end
end)
