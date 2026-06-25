repeat task.wait(0.1) until game:IsLoaded()

-- ════ CARREGAMENTO SEGURO DA RAYFIELD COM MÚLTIPLOS FALLBACKS ════
local Rayfield = nil
local urls = {
    "https://sirius.menu/rayfield",
    "https://raw.githubusercontent.com/SiriusSoftwareLtd/Rayfield/main/source.lua",
    "https://gitlab.com/sirius-software/rayfield/-/raw/main/source.lua",
    "https://cdn.jsdelivr.net/gh/SiriusSoftwareLtd/Rayfield@main/source.lua"
}

print("[Hub] Iniciando carregamento da Rayfield...")
for attempt = 1, 4 do
    for i, url in ipairs(urls) do
        local ok, r = pcall(function() 
            return loadstring(game:HttpGet(url))() 
        end)
        
        if ok and r then 
            Rayfield = r 
            print("[Hub] Rayfield carregada via URL #" .. i .. " (Tentativa " .. attempt .. ")")
            break 
        else
            warn("[Hub] Falha ao carregar URL #" .. i .. " (Tentativa " .. attempt .. ")")
        end
    end
    
    if Rayfield then break end
    
    if attempt < 4 then 
        print("[Hub] Aguardando 5s antes da proxima tentativa...")
        task.wait(5) 
    end
end

if not Rayfield then 
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = game.CoreGui
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 450, 0, 180)
    frame.Position = UDim2.new(0.5, -225, 0.5, -90)
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    frame.BorderSizePixel = 2
    frame.BorderColor3 = Color3.fromRGB(255, 50, 50)
    frame.Parent = screenGui
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.Text = "FALHA CRITICA NO CARREGAMENTO"
    title.TextColor3 = Color3.fromRGB(255, 80, 80)
    title.TextSize = 20
    title.Font = Enum.Font.GothamBold
    title.Parent = frame
    
    local desc = Instance.new("TextLabel")
    desc.Size = UDim2.new(1, -20, 0, 100)
    desc.Position = UDim2.new(0, 10, 0, 45)
    desc.BackgroundTransparency = 1
    desc.Text = "A biblioteca Rayfield nao foi carregada apos 4 tentativas.\n\nMotivo provavel: Erro HTTP 530 (Servidor bloqueado ou instavel).\n\nSolucoes:\n1. Tente reiniciar o jogo\n2. Use outro executor\n3. Ative uma VPN\n4. Aguarde 15-30 minutos"
    desc.TextColor3 = Color3.fromRGB(220, 220, 220)
    desc.TextSize = 16
    desc.Font = Enum.Font.Gotham
    desc.TextWrapped = true
    desc.Parent = frame
    
    error("[Hub] FALHA: Rayfield indisponivel. Verifique o aviso na tela.")
end

-- ═══ CRIAÇÃO DA UI ═══
local Window = Rayfield:CreateWindow({
    Name = "Boat Builder Hub v6",
    LoadingTitle = "Boat Builder",
    LoadingSubtitle = "Build a Boat Edition",
    ConfigurationSaving = { Enabled = false },
    Discord = { Enabled = false },
    KeySystem = false
})

local GoldTab  = Window:CreateTab("Gold Farm", 4483362458)
local BuildTab = Window:CreateTab("Auto Build", 4483362458)
local CodeTab  = Window:CreateTab("Codigos", 4483362458)
local InfoTab  = Window:CreateTab("Inventario", 4483362458)

local Players   = game:GetService("Players")
local RepStor   = game:GetService("ReplicatedStorage")
local Player    = Players.LocalPlayer

-- ════ HELPERS BÁSICOS ═══
local function R()
    local c = Player.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end

local function H()
    local c = Player.Character
    return c and c:FindFirstChild("Humanoid")
end

local function GetGold()
    local d = Player:FindFirstChild("Data")
    local g = d and d:FindFirstChild("Gold")
    return g and g.Value or 0
end

local function notify(t, m, d)
    pcall(function() Rayfield:Notify({Name=t, Content=m, Duration=d or 3}) end)
end

-- ════ SISTEMA DE CÓDIGOS ════
local RedeemRemote = nil

local function FindRedeemRemote()
    if RedeemRemote and RedeemRemote.Parent then return RedeemRemote end
    
    local possibleNames = {"RedeemCode", "ClaimCode", "UseCode", "PromoCode", "Code", "Redeem"}
    for _, name in ipairs(possibleNames) do
        local remote = RepStor:FindFirstChild(name, true)
        if remote and remote:IsA("RemoteEvent") then
            RedeemRemote = remote
            return remote
        end
    end
    
    for _, obj in ipairs(RepStor:GetDescendants()) do
        if obj:IsA("RemoteEvent") then
            local lowerName = obj.Name:lower()
            if lowerName:find("code") or lowerName:find("redeem") or lowerName:find("promo") then
                RedeemRemote = obj
                return obj
            end
        end
    end
    return nil
end

CodeTab:CreateSection("Resgatar Codigos")
CodeTab:CreateLabel("Digite codigos validos abaixo. O script busca o RemoteEvent automaticamente.")

CodeTab:CreateInput({
    Name = "Digite o Codigo",
    PlaceholderText = "Ex: CODE2026",
    RemoveTextAfterFocusLost = false,
    Flag = "CodeInput",
    Callback = function(text)
        if not text or text:gsub("%s+", "") == "" then return end
        local remote = FindRedeemRemote()
        if not remote then
            notify("Erro", "RemoteEvent de codigos nao encontrado.", 5)
            return
        end
        pcall(function()
            remote:FireServer(text)
            notify("Enviado", "Codigo '" .. text .. "' enviado!", 3)
        end)
    end,
})

CodeTab:CreateButton({
    Name = "Forcar Re-deteccao",
    Callback = function()
        RedeemRemote = nil
        local found = FindRedeemRemote()
        if found then
            notify("Sucesso", "Remote encontrado: " .. found.Name, 4)
        else
            notify("Falha", "Nenhum RemoteEvent de codigo encontrado.", 4)
        end
    end,
})

-- ════ PLATAFORMA & GOLD FARM ════
local PLAT_Y     = 25
local PLAT_PAUSE = 4
local CHAR_ABOVE = 6
local platform = nil

local function CreatePlatform(x, z)
    if platform and platform.Parent then pcall(function() platform:Destroy() end) end
    platform = Instance.new("Part")
    platform.Name         = "GoldFarmPlatform"
    platform.Size         = Vector3.new(20, 1, 20)
    platform.Anchored     = true
    platform.CanCollide   = true
    platform.Transparency = 0.4
    platform.BrickColor   = BrickColor.new("Bright yellow")
    platform.Material     = Enum.Material.Neon
    platform.CFrame       = CFrame.new(x or 0, PLAT_Y, z or 0)
    pcall(function() platform.Parent = workspace end)
    return platform
end

local function DestroyPlatform()
    if platform and platform.Parent then
        pcall(function() platform:Destroy() end)
        platform = nil
    end
end

local function TeleportPlatform(x, z)
    if not platform or not platform.Parent then
        platform = CreatePlatform(x, z)
        return
    end
    platform.CFrame = CFrame.new(x, PLAT_Y, z)
end

local function SnapCharToPlatform()
    local r = R()
    if not r or not platform then return end
    r.CFrame = CFrame.new(platform.Position.X, platform.Position.Y + CHAR_ABOVE, platform.Position.Z)
end

_G.GoldFarm  = false
_G.AFKMode   = false
local goldGanho  = 0
local goldInicio = 0
local ciclos     = 0

local function KillPlayer()
    local hum = H()
    if hum then hum.Health = 0 end
end

local function WaitForRespawn()
    local t = 0
    repeat task.wait(0.5) t = t + 0.5
    until R() ~= nil or t >= 10
    task.wait(1.5)
end

local function GetStageFolder()
    local bs = workspace:FindFirstChild("BoatStages")
    if not bs then return nil end
    return bs:FindFirstChild("NormalStages")
end

local function GetStageCenter(stage)
    if not stage then return nil end
    if stage.PrimaryPart then return stage.PrimaryPart.Position end
    local biggest, biggestSize = nil, 0
    for _, v in pairs(stage:GetDescendants()) do
        if v:IsA("BasePart") then
            local s = v.Size.X * v.Size.Z
            if s > biggestSize then biggest=v biggestSize=s end
        end
    end
    return biggest and biggest.Position or nil
end

local function GetChestPos(stage)
    local chest = stage:FindFirstChild("GoldenChest", true)
    if not chest then return nil end
    if chest.PrimaryPart then return chest.PrimaryPart.Position end
    for _, v in pairs(chest:GetDescendants()) do
        if v:IsA("BasePart") then return v.Position end
    end
    if chest:IsA("BasePart") then return chest.Position end
    return nil
end

local function GetRiverPath()
    local folder = GetStageFolder()
    if not folder then return nil, nil end
    local stages = {}
    for _, s in pairs(folder:GetChildren()) do table.insert(stages, s) end
    table.sort(stages, function(a, b)
        local na = tonumber(a.Name:match("%d+")) or 0
        local nb = tonumber(b.Name:match("%d+")) or 0
        return na < nb
    end)
    local waypoints = {}
    for _, stage in ipairs(stages) do
        local pos = GetStageCenter(stage)
        if pos then table.insert(waypoints, Vector3.new(pos.X, PLAT_Y, pos.Z)) end
    end
    return waypoints, stages
end

local function GoldFarmLoop()
    goldInicio = GetGold()
    goldGanho  = 0
    ciclos     = 0
    local waypoints, stages = GetRiverPath()

    if not waypoints or #waypoints == 0 then
        notify("Erro", "Nenhuma fase encontrada!", 5)
        _G.GoldFarm = false
        return
    end

    notify("Gold Farm iniciado!", "Fases: "..#waypoints.."\nModo: Teleporte fase a fase", 5)

    while _G.GoldFarm do
        ciclos = ciclos + 1
        print("[GoldFarm] Ciclo #"..ciclos.." | Ouro: "..GetGold())

        for i, wp in ipairs(waypoints) do
            if not _G.GoldFarm then break end
            local root = R()
            if not root then task.wait(1) continue end

            pcall(function()
                TeleportPlatform(wp.X, wp.Z)
                task.wait(0.05)
                SnapCharToPlatform()
                task.wait(0.1)
            end)

            print("[GoldFarm] Fase "..i.."/"..#waypoints)

            local pauseStart = tick()
            while tick() - pauseStart < PLAT_PAUSE do
                if not _G.GoldFarm then break end
                local r = R()
                if r then
                    r.CFrame = CFrame.new(platform.Position.X, platform.Position.Y + CHAR_ABOVE, platform.Position.Z)
                end
                task.wait(0.1)
            end

            local goldAtual   = GetGold()
            local ganhouAgora = goldAtual - (goldInicio + goldGanho)
            if ganhouAgora > 0 then
                goldGanho = goldGanho + ganhouAgora
                print("[GoldFarm] +"..ganhouAgora.." ouro na fase "..i)
            end

            if stages and stages[i] then
                local chestPos = GetChestPos(stages[i])
                if chestPos then
                    pcall(function()
                        TeleportPlatform(chestPos.X, chestPos.Z)
                        task.wait(0.05)
                        SnapCharToPlatform()
                    end)
                    local chestStart = tick()
                    while tick() - chestStart < 2 do
                        if not _G.GoldFarm then break end
                        local r = R()
                        if r then
                            r.CFrame = CFrame.new(platform.Position.X, platform.Position.Y + CHAR_ABOVE, platform.Position.Z)
                        end
                        task.wait(0.1)
                    end
                    local goldBau = GetGold() - (goldInicio + goldGanho)
                    if goldBau > 0 then
                        goldGanho = goldGanho + goldBau
                        print("[GoldFarm] Bau +"..goldBau)
                    end
                end
            end
            task.wait(0.05)
        end

        if not _G.GoldFarm then break end
        notify("Reiniciando", "Ciclo #"..ciclos.." completo! +"..goldGanho.." ouro", 3)
        KillPlayer()
        task.wait(0.5)
        DestroyPlatform()
        WaitForRespawn()
        if _G.GoldFarm and waypoints[1] then
            CreatePlatform(waypoints[1].X, waypoints[1].Z)
            task.wait(1)
        end
    end

    DestroyPlatform()
    notify("Farm parado", "Ciclos: "..ciclos.."\nOuro ganho: +"..goldGanho, 6)
end

local function AFKLoop()
    notify("AFK", "Anti-kick ativado.", 3)
    local t = 0
    while _G.AFKMode do
        task.wait(0.5)
        t = t + 0.5
        if t % 55 < 0.5 then
            local hum = H()
            if hum and hum.Health > 0 then hum.Jump = true end
        end
    end
end

-- ════ UI: GOLD TAB ═══
GoldTab:CreateSection("Farm - Teleporte Fase a Fase")
GoldTab:CreateToggle({
    Name = "Ativar Gold Farm", CurrentValue = false, Flag = "GF",
    Callback = function(v) _G.GoldFarm = v; if v then task.spawn(GoldFarmLoop) else DestroyPlatform() end end,
})
GoldTab:CreateSlider({
    Name = "Pausa por Fase (segundos)", Range = {1, 15}, Increment = 1, Suffix = "s",
    CurrentValue = 4, Flag = "PauseTime", Callback = function(v) PLAT_PAUSE = v end,
})
GoldTab:CreateSlider({
    Name = "Altura da Plataforma", Range = {10, 60}, Increment = 5, Suffix = " studs",
    CurrentValue = 25, Flag = "PlatHeight", Callback = function(v) PLAT_Y = v end,
})
GoldTab:CreateButton({
    Name = "Estatisticas",
    Callback = function()
        notify("Stats", "Ouro: "..GetGold().."\nGanho: +"..goldGanho.."\nCiclos: "..ciclos, 6)
    end,
})
GoldTab:CreateButton({
    Name = "Destruir Plataforma",
    Callback = function() DestroyPlatform(); notify("Ok","Plataforma destruida.",2) end,
})
GoldTab:CreateSection("AFK Anti-Kick")
GoldTab:CreateToggle({
    Name = "Ativar AFK", CurrentValue = false, Flag = "AFK",
    Callback = function(v) _G.AFKMode = v; if v then task.spawn(AFKLoop) end end,
})

-- ════ BUILD TAB CORRIGIDA PARA MOBILE ═══
local BoatBlocks = {}
for x=-4,4 do for z=0,8 do
    table.insert(BoatBlocks,{x,0,z}); table.insert(BoatBlocks,{x,1,z})
end end
for z=0,8 do for y=2,5 do
    table.insert(BoatBlocks,{-4,y,z}); table.insert(BoatBlocks,{-3,y,z})
    table.insert(BoatBlocks,{4,y,z}); table.insert(BoatBlocks,{3,y,z})
end end
for x=-4,4 do for y=2,5 do
    table.insert(BoatBlocks,{x,y,0}); table.insert(BoatBlocks,{x,y,9})
end end
for x=-4,4 do for z=0,4 do
    table.insert(BoatBlocks,{x,6,z})
end end

local TOTAL = #BoatBlocks
_G.Building      = false
_G.BuildSpeed    = 0.08
local builtCount = 0
local buildRemote = nil

local function GetBuildRemote()
    if buildRemote and buildRemote.Parent then return buildRemote end
    
    local exactNames = {"QueueBlocksRequest", "PlaceBlock", "BuildBlock", "Construct"}
    for _, name in ipairs(exactNames) do
        local remote = RepStor:FindFirstChild(name, true)
        if remote and remote:IsA("RemoteEvent") then
            buildRemote = remote
            return remote
        end
    end
    
    for _, obj in ipairs(RepStor:GetDescendants()) do
        if obj:IsA("RemoteEvent") then
            local n = obj.Name:lower()
            if n:find("block") or n:find("queue") or n:find("place") or n:find("build") then
                buildRemote = obj
                return obj
            end
        end
    end
    return nil
end

local function GetBasePosition()
    local root = R()
    if not root then return nil end
    return Vector3.new(root.Position.X - 12, 3, root.Position.Z - 4)
end

local function CheckInventoryForBuild()
    local data = Player:FindFirstChild("Data")
    if not data then return false, "Pasta Data nao encontrada." end
    
    local woodBlockValue = nil
    local possibleNames = {"WoodBlock", "Wood", "BasicBlock", "Block_Wood"}
    
    for _, name in ipairs(possibleNames) do
        local val = data:FindFirstChild(name)
        if val and (val:IsA("IntValue") or val:IsA("NumberValue")) then
            woodBlockValue = val
            break
        end
    end
    
    if not woodBlockValue then
        for _, child in ipairs(data:GetChildren()) do
            if (child:IsA("IntValue") or child:IsA("NumberValue")) then
                local lowerName = child.Name:lower()
                if lowerName:find("wood") and lowerName:find("block") then
                    woodBlockValue = child
                    break
                end
            end
        end
    end
    
    if not woodBlockValue then
        return false, "Nao foi possivel identificar estoque de WoodBlocks."
    end
    
    local available = woodBlockValue.Value
    if available < TOTAL then
        return false, string.format(
            "Blocos insuficientes! Disponiveis: %d | Necessarios: %d | Faltam: %d",
            available, TOTAL, TOTAL - available
        )
    end
    
    return true, string.format("%d WoodBlocks disponiveis", available)
end

local function PlaceBlock(relX, relY, relZ)
    local remote = GetBuildRemote()
    if not remote then return false end
    
    local base = GetBasePosition()
    if not base then return false end
    
    local worldPos = Vector3.new(base.X + relX * 3, base.Y + relY * 3, base.Z + relZ * 3)
    
    local success = pcall(function()
        remote:FireServer({
            Action = "Place",
            BlockType = "WoodBlock",
            Position = worldPos,
            Rotation = Vector3.new(0, 0, 0),
            Color = "Medium stone grey"
        })
    end)
    return success
end

BuildTab:CreateSection("Barco Resistente")
BuildTab:CreateLabel("Total: "..TOTAL.." WoodBlocks necessarios")
BuildTab:CreateLabel("Construi proximo ao spawn (Y=3)")

local statusLabel = BuildTab:CreateLabel("Status: Pronto para construir")
local progressLabel = BuildTab:CreateLabel("Progresso: 0/"..TOTAL)

BuildTab:CreateButton({
    Name = "Construir Barco",
    Callback = function()
        if _G.Building then 
            notify("Atencao","Construcao ja em andamento!",2) 
            return 
        end
        
        local canBuild, message = CheckInventoryForBuild()
        if not canBuild then
            notify("Bloqueado", message, 6)
            pcall(function() statusLabel:SetText("Status: Bloqueado - " .. message:sub(1, 30)) end)
            return
        end
        
        local remote = GetBuildRemote()
        if not remote then
            notify("Erro","RemoteEvent de construcao nao encontrado.",5)
            pcall(function() statusLabel:SetText("Status: Remote nao detectado") end)
            return
        end
        
        _G.Building = true
        builtCount  = 0
        pcall(function() statusLabel:SetText("Status: Construindo...") end)
        notify("Build Iniciado", message .. "\nColocando "..TOTAL.." blocos...", 4)
        
        task.spawn(function()
            local failed = 0
            local lastUpdate = tick()
            
            for i, b in ipairs(BoatBlocks) do
                if not _G.Building then break end
                
                local ok = PlaceBlock(b[1], b[2], b[3])
                if ok then
                    builtCount = builtCount + 1
                else
                    failed = failed + 1
                end
                
                if i % 10 == 0 or (tick() - lastUpdate) > 0.5 then
                    pcall(function()
                        progressLabel:SetText(
                            string.format("Progresso: %d/%d | Falhas: %d (%.1f%%)",
                                builtCount, TOTAL, failed, (builtCount/TOTAL)*100)
                        )
                    end)
                    lastUpdate = tick()
                end
                
                task.wait(_G.BuildSpeed)
            end
            
            if _G.Building then
                _G.Building = false
                pcall(function()
                    progressLabel:SetText(string.format("Concluido: %d/%d | Falhas: %d", builtCount, TOTAL, failed))
                    statusLabel:SetText("Status: Construcao finalizada")
                end)
                notify("Pronto!", string.format("Barco construido!\nColocados: %d/%d\nFalhas: %d", builtCount, TOTAL, failed), 6)
            end
        end)
    end,
})

BuildTab:CreateButton({
    Name = "Parar Build",
    Callback = function() 
      
