-- Script único Kick GUI para executor Roblox (LocalScript)
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui") -- garante que o PlayerGui existe

-- Lista de admins (coloque seu UserId e de quem quiser)
local AdminUserIds = {
    [player.UserId] = true, -- você mesmo
}

-- Checar se é admin
local function isAdmin(userId)
    return AdminUserIds[userId] == true
end

if not isAdmin(player.UserId) then
    warn("Você não é admin e não pode usar este script.")
    return
end

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KickGuiTest"
ScreenGui.Parent = playerGui

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 350, 0, 400)
Frame.Position = UDim2.new(0, 20, 0, 50)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.BackgroundTransparency = 0.1

-- Título
local Title = Instance.new("TextLabel", Frame)
Title.Text = "Kick Test"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20

-- Caixa para nome do jogador
local targetBox = Instance.new("TextBox", Frame)
targetBox.PlaceholderText = "Nome do jogador"
targetBox.Size = UDim2.new(1, -20, 0, 30)
targetBox.Position = UDim2.new(0, 10, 0, 50)

-- Caixa para motivo
local reasonBox = Instance.new("TextBox", Frame)
reasonBox.PlaceholderText = "Motivo (opcional)"
reasonBox.Size = UDim2.new(1, -20, 0, 30)
reasonBox.Position = UDim2.new(0, 10, 0, 90)

-- Botão de kick simulado
local kickButton = Instance.new("TextButton", Frame)
kickButton.Text = "Simular Kick"
kickButton.Size = UDim2.new(1, -20, 0, 30)
kickButton.Position = UDim2.new(0, 10, 0, 130)
kickButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
kickButton.TextColor3 = Color3.fromRGB(255, 255, 255)
kickButton.Font = Enum.Font.SourceSansBold
kickButton.TextSize = 18

-- Título da lista de jogadores
local listTitle = Instance.new("TextLabel", Frame)
listTitle.Text = "Jogadores Online"
listTitle.Size = UDim2.new(1, 0, 0, 20)
listTitle.Position = UDim2.new(0, 0, 0, 170)
listTitle.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
listTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
listTitle.Font = Enum.Font.SourceSans
listTitle.TextSize = 16

-- Frame da lista
local listFrame = Instance.new("ScrollingFrame", Frame)
listFrame.Size = UDim2.new(1, -20, 0, 180)
listFrame.Position = UDim2.new(0, 10, 0, 195)
listFrame.CanvasSize = UDim2.new(0,0,0,0)
listFrame.ScrollBarThickness = 6
listFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
listFrame.BorderSizePixel = 0

local UIListLayout = Instance.new("UIListLayout", listFrame)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0,5)

-- Atualizar lista de jogadores
local function updatePlayerList()
    listFrame:ClearAllChildren()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            local btn = Instance.new("TextButton", listFrame)
            btn.Size = UDim2.new(1, 0, 0, 25)
            btn.Text = plr.Name
            btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
            btn.TextColor3 = Color3.fromRGB(255,255,255)
            btn.Font = Enum.Font.SourceSans
            btn.TextSize = 16
            btn.MouseButton1Click:Connect(function()
                targetBox.Text = plr.Name
            end)
        end
    end
    listFrame.CanvasSize = UDim2.new(0,0,0, UIListLayout.AbsoluteContentSize.Y)
end

Players.PlayerAdded:Connect(updatePlayerList)
Players.PlayerRemoving:Connect(updatePlayerList)
updatePlayerList()

-- Função de kick simulado
kickButton.MouseButton1Click:Connect(function()
    local targetName = targetBox.Text
    local reason = reasonBox.Text or "Kick de teste"
    local target = Players:FindFirstChild(targetName)
    
    if not target then
        kickButton.Text = "Jogador não encontrado"
        task.wait(1.5)
        kickButton.Text = "Simular Kick"
        return
    end

    if AdminUserIds[target.UserId] then
        kickButton.Text = "Não pode simular kick em admin"
        task.wait(1.5)
        kickButton.Text = "Simular Kick"
        return
    end

    print(string.format("[TEST KICK] %s simulou kick em %s. Motivo: %s", player.Name, target.Name, reason))
    kickButton.Text = target.Name .. " simulado!"
    task.wait(2)
    kickButton.Text = "Simular Kick"
end)
