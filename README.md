-- Script seguro: faz jogador seguir você (ficar na sua frente)
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Lista de admins
local AdminUserIds = {
    [player.UserId] = true
}

local function isAdmin(userId)
    return AdminUserIds[userId] == true
end

if not isAdmin(player.UserId) then
    warn("Você não é admin e não pode usar este script.")
    return
end

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FollowGui"
ScreenGui.Parent = playerGui

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 350, 0, 400)
Frame.Position = UDim2.new(0, 20, 0, 50)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.BackgroundTransparency = 0.1

local Title = Instance.new("TextLabel", Frame)
Title.Text = "Follow Test"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20

-- Lista de jogadores
local listFrame = Instance.new("ScrollingFrame", Frame)
listFrame.Size = UDim2.new(1, -20, 0, 300)
listFrame.Position = UDim2.new(0, 10, 0, 60)
listFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
listFrame.BorderSizePixel = 0
listFrame.CanvasSize = UDim2.new(0,0,0,0)

local UIListLayout = Instance.new("UIListLayout", listFrame)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0,5)

-- Atualizar lista
local function updatePlayerList()
    listFrame:ClearAllChildren()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            local btn = Instance.new("TextButton", listFrame)
            btn.Size = UDim2.new(1,0,0,25)
            btn.Text = plr.Name
            btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
            btn.TextColor3 = Color3.fromRGB(255,255,255)
            btn.Font = Enum.Font.SourceSans
            btn.TextSize = 16

            btn.MouseButton1Click:Connect(function()
                local character = plr.Character
                local myChar = player.Character
                if character and character:FindFirstChild("HumanoidRootPart") and myChar and myChar:FindFirstChild("HumanoidRootPart") then
                    print("[FOLLOW] "..plr.Name.." agora vai ficar na sua frente.")
                    -- Teletransporta o jogador repetidamente para frente de você
                    local followConnection
                    followConnection = game:GetService("RunService").RenderStepped:Connect(function()
                        if character.Parent and myChar.Parent then
                            local offset = myChar.HumanoidRootPart.CFrame.LookVector * 3
                            character.HumanoidRootPart.CFrame = myChar.HumanoidRootPart.CFrame + offset
                        else
                            followConnection:Disconnect()
                        end
                    end)
                end
            end)
        end
    end
    listFrame.CanvasSize = UDim2.new(0,0,0, UIListLayout.AbsoluteContentSize.Y)
end

Players.PlayerAdded:Connect(updatePlayerList)
Players.PlayerRemoving:Connect(updatePlayerList)
updatePlayerList()
