-- Script de servidor para “seguir você” (visível para todos)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Tabela para guardar conexões de follow
local followConnections = {}

-- Função para iniciar o follow
local function startFollow(targetPlayer, referencePlayer)
    if not targetPlayer.Character or not referencePlayer.Character then return end
    local targetHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    local refHRP = referencePlayer.Character:FindFirstChild("HumanoidRootPart")
    if not targetHRP or not refHRP then return end

    -- Se já existir conexão, desconecta primeiro
    if followConnections[targetPlayer] then
        followConnections[targetPlayer]:Disconnect()
    end

    -- Criar nova conexão
    followConnections[targetPlayer] = RunService.Heartbeat:Connect(function()
        if targetPlayer.Character and targetHRP.Parent and refHRP.Parent then
            local offset = refHRP.CFrame.LookVector * 3
            targetHRP.CFrame = refHRP.CFrame + offset
        else
            followConnections[targetPlayer]:Disconnect()
            followConnections[targetPlayer] = nil
        end
    end)
    print("[FOLLOW] "..targetPlayer.Name.." agora está seguindo "..referencePlayer.Name)
end

-- Função para parar o follow
local function stopFollow(targetPlayer)
    if followConnections[targetPlayer] then
        followConnections[targetPlayer]:Disconnect()
        followConnections[targetPlayer] = nil
        print("[FOLLOW] "..targetPlayer.Name.." parou de seguir")
    end
end

-- Exemplo de uso:
-- Substitua "JogadorAlvo" e "Você" pelos nomes reais dos jogadores no servidor
local jogadorAlvo = Players:FindFirstChild("JogadorAlvo") -- jogador que vai seguir
local voce = Players:FindFirstChild("Você") -- jogador de referência

if jogadorAlvo and voce then
    startFollow(jogadorAlvo, voce) -- começa a seguir
    -- Para parar, chame:
    -- stopFollow(jogadorAlvo)
end
