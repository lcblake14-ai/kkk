-- Script completo: GUI + RemoteEvent + Follow para testes (ServerScriptService)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- Criar RemoteEvent se não existir
local followEvent = ReplicatedStorage:FindFirstChild("FollowEvent")
if not followEvent then
    followEvent = Instance.new("RemoteEvent")
    followEvent.Name = "FollowEvent"
    followEvent.Parent = ReplicatedStorage
end

-- Tabela para armazenar follow connections
local followConnections = {}

-- Função do servidor para mover jogador
followEvent.OnServerEvent:Connect(function(player, targetPlayerName, action)
    local targetPlayer = Players:FindFirstChild(targetPlayerName)
    if not targetPlayer or not targetPlayer.Character or not player.Character then return end

    local targetHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    local playerHRP = player.Character:FindFirstChild("HumanoidRootPart")
    if not targetHRP or not playerHRP then return end

    if action == "start" then
        if followConnections[targetPlayer] then
            followConnections[targetPlayer]:Disconnect()
        end
        followConnections[targetPlayer] = RunService.Heartbeat:Connect(function()
            if targetPlayer.Character and targetHRP.Parent and playerHRP.Parent then
                local offset = playerHRP.CFrame.LookVector * 3
                targetHRP.CFrame = playerHRP.CFrame + offset
            else
                followConnections[targetPlayer]:Disconnect()
                followConnections[targetPlayer] = nil
            end
        end)
        print("[FOLLOW] "..targetPlayer.Name.." está seguindo "..player.Name)
    elseif action == "stop" then
        if followConnections[targetPlayer] then
            followConnections[targetPlayer]:Disconnect()
            followConnections[targetPlayer] = nil
            print("[FOLLOW] "..targetPlayer.Name.." parou de seguir")
        end
    end
end)

-- Criar GUI para todos os jogadores
Players.PlayerAdded:Connect(function(player)
    local playerGui = player:WaitForChild("PlayerGui")
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "FollowGui"
    ScreenGui.Parent = playerGui

    local Frame = Instance.new("Frame", ScreenGui)
    Frame.Size = UDim2.new(0, 350, 0, 450)
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

    -- Barra de pesquisa
    local searchBox = Instance.new("TextBox", Frame)
    searchBox.PlaceholderText = "Pesquisar jogador..."
    searchBox.Size = UDim2.new(1, -20, 0, 30)
    searchBox.Position = UDim2.new(0, 10, 0, 40)

    -- Lista de jogadores
    local listFrame = Instance.new("ScrollingFrame", Frame)
    listFrame.Size = UDim2.new(1, -20, 0, 350)
    listFrame.Position = UDim2.new(0, 10, 0, 80)
    listFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
    listFrame.BorderSizePixel = 0
    listFrame.CanvasSize = UDim2.new(0,0,0,0)

    local UIListLayout = Instance.new("UIListLayout", listFrame)
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UIListLayout.Padding = UDim.new(0,5)

    local function createPlayerButtons(playersToShow)
        listFrame:ClearAllChildren()
        for _, plr in pairs(playersToShow) do
            if plr ~= player then
                local btn = Instance.new("TextButton", listFrame)
                btn.Size = UDim2.new(1,0,0,25)
                btn.Text = plr.Name
                btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
                btn.TextColor3 = Color3.fromRGB(255,255,255)
                btn.Font = Enum.Font.SourceSans
                btn.TextSize = 16

                local following = false

                btn.MouseButton1Click:Connect(function()
                    if not following then
                        followEvent:FireServer(plr.Name, "start")
                        btn.Text = plr.Name.." [Seguindo]"
                        following = true
                    else
                        followEvent:FireServer(plr.Name, "stop")
                        btn.Text = plr.Name
                        following = false
                    end
                end)
            end
        end
        listFrame.CanvasSize = UDim2.new(0,0,0, UIListLayout.AbsoluteContentSize.Y)
    end

    local function updatePlayerList()
        createPlayerButtons(Players:GetPlayers())
    end

    Players.PlayerAdded:Connect(updatePlayerList)
    Players.PlayerRemoving:Connect(updatePlayerList)
    updatePlayerList()

    searchBox:GetPropertyChangedSignal("Text"):Connect(function()
        local text = searchBox.Text:lower()
        local filtered = {}
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Name:lower():find(text) then
                table.insert(filtered, plr)
            end
        end
        createPlayerButtons(filtered)
    end)
end)
