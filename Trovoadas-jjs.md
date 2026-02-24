-- EB JJs Xeno - Exército Brasileiro
-- Formato: UM !, DOIS !, TRÊS ! ...
-- Compatível com Xeno Executor

local Players         = game:GetService("Players")
local RunService      = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService    = game:GetService("TweenService")
local LocalPlayer     = Players.LocalPlayer

-- CONFIGURAÇÕES
local Config = {
    Tempo      = 2.5,
    NumInicio  = 1,
    NumFim     = 50,
    PularJunto = false,
    Rainbow    = false,
    Rodando    = false,
}

-- CONVERSÃO NÚMERO → EXTENSO EM MAIÚSCULAS
local function extenso(n)
    local uns = {"","UM","DOIS","TRÊS","QUATRO","CINCO","SEIS","SETE","OITO","NOVE",
        "DEZ","ONZE","DOZE","TREZE","QUATORZE","QUINZE","DEZESSEIS","DEZESSETE","DEZOITO","DEZENOVE"}
    local dez = {"","","VINTE","TRINTA","QUARENTA","CINQUENTA","SESSENTA","SETENTA","OITENTA","NOVENTA"}
    local cen = {"","CEM","DUZENTOS","TREZENTOS","QUATROCENTOS","QUINHENTOS",
        "SEISCENTOS","SETECENTOS","OITOCENTOS","NOVECENTOS"}

    n = math.floor(n)
    if n == 0 then return "ZERO" end

    local p = {}

    if n >= 1000 then
        local m = math.floor(n / 1000)
        table.insert(p, m == 1 and "MIL" or (extenso(m) .. " MIL"))
        n = n % 1000
    end
    if n >= 100 then
        local c = math.floor(n / 100)
        table.insert(p, (n % 100 == 0) and cen[c] or (c == 1 and "CENTO" or cen[c]))
        n = n % 100
    end
    if n >= 20 then
        local d = math.floor(n / 10)
        local u = n % 10
        table.insert(p, u == 0 and dez[d] or (dez[d] .. " E " .. uns[u + 1]))
    elseif n > 0 then
        table.insert(p, uns[n + 1])
    end

    return table.concat(p, " E ")
end

-- ENVIO NO CHAT (Xeno)
local function enviarChat(msg)
    -- Tenta todos os TextChannels disponíveis
    local function tentarTextChatService()
        local TCS = game:GetService("TextChatService")
        local channels = TCS:FindFirstChild("TextChannels")
        if not channels then return false end
        -- tenta RBXGeneral primeiro, depois qualquer channel
        local targets = {
            channels:FindFirstChild("RBXGeneral"),
            channels:FindFirstChild("RBXSystem"),
            channels:FindFirstChildOfClass("TextChannel"),
        }
        for _, ch in ipairs(targets) do
            if ch then
                local ok = pcall(function() ch:SendAsync(msg) end)
                if ok then return true end
            end
        end
        return false
    end

    -- Tenta SayMessageRequest (chat legado)
    local function tentarLegado()
        local ev = game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents")
        if not ev then return false end
        local say = ev:FindFirstChild("SayMessageRequest")
        if not say then return false end
        local ok = pcall(function() say:FireServer(msg, "All") end)
        return ok
    end

    -- Tenta todos os RemoteEvents com nome relacionado a chat
    local function tentarRemotes()
        for _, v in pairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
            if v:IsA("RemoteEvent") then
                local nome = v.Name:lower()
                if nome:find("say") or nome:find("chat") or nome:find("message") or nome:find("talk") then
                    local ok = pcall(function() v:FireServer(msg, "All") end)
                    if ok then return true end
                end
            end
        end
        return false
    end

    if tentarTextChatService() then return end
    if tentarLegado() then return end
    tentarRemotes()
end

local function pular()
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
end

-- LOOP JJs
local loopThread = nil

local function iniciar(onUpdate)
    Config.Rodando = true
    loopThread = task.spawn(function()
        local i = Config.NumInicio
        while Config.Rodando and i <= Config.NumFim do
            local msg = extenso(i) .. " !"
            enviarChat(msg)
            if Config.PularJunto then pular() end
            if onUpdate then onUpdate(i) end
            i = i + 1
            task.wait(Config.Tempo)
        end
        Config.Rodando = false
        if onUpdate then onUpdate(nil) end
    end)
end

local function parar()
    Config.Rodando = false
    if loopThread then
        task.cancel(loopThread)
        loopThread = nil
    end
end

-- =============================================
-- GUI
-- =============================================
local oldGui = game:GetService("CoreGui"):FindFirstChild("EB_JJs_Xeno")
if oldGui then oldGui:Destroy() end

local SG = Instance.new("ScreenGui")
SG.Name = "EB_JJs_Xeno"
SG.ResetOnSpawn = false
SG.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
SG.Parent = game:GetService("CoreGui")

-- Janela principal
local Win = Instance.new("Frame", SG)
Win.Size = UDim2.new(0, 360, 0, 450)
Win.Position = UDim2.new(0.5, -180, 0.5, -225)
Win.BackgroundColor3 = Color3.fromRGB(13, 14, 22)
Win.BorderSizePixel = 0
Win.Active = true
Win.Draggable = true
Instance.new("UICorner", Win).CornerRadius = UDim.new(0, 12)

local WinStroke = Instance.new("UIStroke", Win)
WinStroke.Color = Color3.fromRGB(0, 200, 80)
WinStroke.Thickness = 1.5

-- Topo
local Top = Instance.new("Frame", Win)
Top.Size = UDim2.new(1, 0, 0, 48)
Top.BackgroundColor3 = Color3.fromRGB(15, 130, 55)
Top.BorderSizePixel = 0
Instance.new("UICorner", Top).CornerRadius = UDim.new(0, 12)
-- fix cantos inferiores do topo
local topFix = Instance.new("Frame", Top)
topFix.Size = UDim2.new(1, 0, 0, 12)
topFix.Position = UDim2.new(0, 0, 1, -12)
topFix.BackgroundColor3 = Color3.fromRGB(15, 130, 55)
topFix.BorderSizePixel = 0

-- Título
local TitleLbl = Instance.new("TextLabel", Top)
TitleLbl.Size = UDim2.new(1, -90, 0, 26)
TitleLbl.Position = UDim2.new(0, 10, 0, 4)
TitleLbl.BackgroundTransparency = 1
TitleLbl.Text = "🪖  AutoJJs | Exército Brasileiro"
TitleLbl.TextSize = 13
TitleLbl.Font = Enum.Font.GothamBold
TitleLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLbl.TextXAlignment = Enum.TextXAlignment.Left

local SubLbl = Instance.new("TextLabel", Top)
SubLbl.Size = UDim2.new(1, -90, 0, 14)
SubLbl.Position = UDim2.new(0, 10, 0, 30)
SubLbl.BackgroundTransparency = 1
SubLbl.Text = "Xeno Edition — HOME para abrir/fechar"
SubLbl.TextSize = 9
SubLbl.Font = Enum.Font.Gotham
SubLbl.TextColor3 = Color3.fromRGB(180, 255, 180)
SubLbl.TextXAlignment = Enum.TextXAlignment.Left

-- Botão fechar
local BClose = Instance.new("TextButton", Top)
BClose.Size = UDim2.new(0, 30, 0, 30)
BClose.Position = UDim2.new(1, -36, 0.5, -15)
BClose.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
BClose.Text = "✕"
BClose.TextSize = 12
BClose.Font = Enum.Font.GothamBold
BClose.TextColor3 = Color3.fromRGB(255, 255, 255)
BClose.BorderSizePixel = 0
Instance.new("UICorner", BClose).CornerRadius = UDim.new(0, 6)
BClose.MouseButton1Click:Connect(function() SG:Destroy() end)

-- Botão minimizar
local BMin = Instance.new("TextButton", Top)
BMin.Size = UDim2.new(0, 30, 0, 30)
BMin.Position = UDim2.new(1, -70, 0.5, -15)
BMin.BackgroundColor3 = Color3.fromRGB(180, 130, 0)
BMin.Text = "─"
BMin.TextSize = 12
BMin.Font = Enum.Font.GothamBold
BMin.TextColor3 = Color3.fromRGB(255, 255, 255)
BMin.BorderSizePixel = 0
Instance.new("UICorner", BMin).CornerRadius = UDim.new(0, 6)

-- Corpo
local Body = Instance.new("Frame", Win)
Body.Size = UDim2.new(1, 0, 1, -48)
Body.Position = UDim2.new(0, 0, 0, 48)
Body.BackgroundTransparency = 1

local BLayout = Instance.new("UIListLayout", Body)
BLayout.Padding = UDim.new(0, 8)
BLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

local BPad = Instance.new("UIPadding", Body)
BPad.PaddingTop    = UDim.new(0, 12)
BPad.PaddingLeft   = UDim.new(0, 12)
BPad.PaddingRight  = UDim.new(0, 12)
BPad.PaddingBottom = UDim.new(0, 12)

BMin.MouseButton1Click:Connect(function()
    Body.Visible = not Body.Visible
    Win.Size = Body.Visible and UDim2.new(0, 360, 0, 450) or UDim2.new(0, 360, 0, 48)
end)

-- Helper card
local function Card(h)
    local f = Instance.new("Frame", Body)
    f.Size = UDim2.new(1, 0, 0, h or 44)
    f.BackgroundColor3 = Color3.fromRGB(20, 21, 32)
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    return f
end

-- ── Intervalo de tempo ────────────────────────────────────────
do
    local c = Card(44)
    local lbl = Instance.new("TextLabel", c)
    lbl.Size = UDim2.new(0.6, 0, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = "⏱  Intervalo (segundos)"
    lbl.TextSize = 11
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextColor3 = Color3.fromRGB(160, 160, 180)
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local val = Instance.new("TextLabel", c)
    val.Size = UDim2.new(0, 36, 0, 22)
    val.Position = UDim2.new(1, -44, 0.5, -11)
    val.BackgroundTransparency = 1
    val.Text = tostring(Config.Tempo)
    val.TextSize = 13
    val.Font = Enum.Font.GothamBold
    val.TextColor3 = Color3.fromRGB(60, 220, 120)
    val.TextXAlignment = Enum.TextXAlignment.Center

    local function mkBtn(xoff, sym, cor, delta)
        local b = Instance.new("TextButton", c)
        b.Size = UDim2.new(0, 24, 0, 24)
        b.Position = UDim2.new(1, xoff, 0.5, -12)
        b.BackgroundColor3 = cor
        b.Text = sym
        b.TextSize = 14
        b.Font = Enum.Font.GothamBold
        b.TextColor3 = Color3.fromRGB(255, 255, 255)
        b.BorderSizePixel = 0
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)
        b.MouseButton1Click:Connect(function()
            Config.Tempo = math.clamp(Config.Tempo + delta, 0.5, 30)
            val.Text = string.format("%.1f", Config.Tempo)
        end)
    end
    mkBtn(-86, "+", Color3.fromRGB(20, 150, 60),  0.5)
    mkBtn(-60, "−", Color3.fromRGB(170, 30, 30), -0.5)
end

-- ── Intervalo de números ──────────────────────────────────────
do
    local c = Card(76)

    local hdr = Instance.new("TextLabel", c)
    hdr.Size = UDim2.new(1, -10, 0, 22)
    hdr.Position = UDim2.new(0, 10, 0, 4)
    hdr.BackgroundTransparency = 1
    hdr.Text = "🔢  Intervalo de Números"
    hdr.TextSize = 11
    hdr.Font = Enum.Font.GothamSemibold
    hdr.TextColor3 = Color3.fromRGB(160, 160, 180)
    hdr.TextXAlignment = Enum.TextXAlignment.Left

    local function mkBox(label, xpos, cfgKey, default)
        local L = Instance.new("TextLabel", c)
        L.Size = UDim2.new(0, 30, 0, 26)
        L.Position = UDim2.new(0, xpos, 0, 36)
        L.BackgroundTransparency = 1
        L.Text = label
        L.TextSize = 11
        L.Font = Enum.Font.Gotham
        L.TextColor3 = Color3.fromRGB(150, 150, 150)
        L.TextXAlignment = Enum.TextXAlignment.Left

        local box = Instance.new("TextBox", c)
        box.Size = UDim2.new(0, 70, 0, 26)
        box.Position = UDim2.new(0, xpos + 32, 0, 36)
        box.BackgroundColor3 = Color3.fromRGB(28, 30, 46)
        box.BorderSizePixel = 0
        box.Text = tostring(default)
        box.TextSize = 12
        box.Font = Enum.Font.GothamBold
        box.TextColor3 = Color3.fromRGB(60, 220, 120)
        box.ClearTextOnFocus = false
        Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6)
        box.FocusLost:Connect(function()
            Config[cfgKey] = tonumber(box.Text) or default
            box.Text = tostring(Config[cfgKey])
        end)
    end

    mkBox("De:",  10,  "NumInicio", 1)
    mkBox("Até:", 170, "NumFim",    50)
end

-- ── Toggles ──────────────────────────────────────────────────
local function mkToggle(labelTxt, cfgKey)
    local c = Card(38)

    local lbl = Instance.new("TextLabel", c)
    lbl.Size = UDim2.new(1, -60, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = labelTxt
    lbl.TextSize = 12
    lbl.Font = Enum.Font.Gotham
    lbl.TextColor3 = Color3.fromRGB(200, 200, 200)
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local bg = Instance.new("Frame", c)
    bg.Size = UDim2.new(0, 40, 0, 22)
    bg.Position = UDim2.new(1, -48, 0.5, -11)
    bg.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    bg.BorderSizePixel = 0
    Instance.new("UICorner", bg).CornerRadius = UDim.new(1, 0)

    local dot = Instance.new("Frame", bg)
    dot.Size = UDim2.new(0, 16, 0, 16)
    dot.Position = UDim2.new(0, 3, 0.5, -8)
    dot.BackgroundColor3 = Color3.fromRGB(130, 130, 130)
    dot.BorderSizePixel = 0
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local state = false
    local ti = TweenInfo.new(0.15)

    local function refresh()
        if state then
            TweenService:Create(bg,  ti, {BackgroundColor3 = Color3.fromRGB(20, 180, 70)}):Play()
            TweenService:Create(dot, ti, {Position = UDim2.new(0, 21, 0.5, -8), BackgroundColor3 = Color3.fromRGB(255, 255, 255)}):Play()
        else
            TweenService:Create(bg,  ti, {BackgroundColor3 = Color3.fromRGB(50, 50, 65)}):Play()
            TweenService:Create(dot, ti, {Position = UDim2.new(0, 3, 0.5, -8),  BackgroundColor3 = Color3.fromRGB(130, 130, 130)}):Play()
        end
    end

    local hit = Instance.new("TextButton", c)
    hit.Size = UDim2.new(1, 0, 1, 0)
    hit.BackgroundTransparency = 1
    hit.Text = ""
    hit.MouseButton1Click:Connect(function()
        state = not state
        Config[cfgKey] = state
        refresh()
    end)

    refresh()
end

mkToggle("🦘  Pular junto ao enviar JJ", "PularJunto")
mkToggle("🌈  Interface arco-íris",      "Rainbow")

-- ── Botão TESTE (envia 1 mensagem pra confirmar chat) ─────────
do
    local c = Card(36)
    local btn = Instance.new("TextButton", c)
    btn.Size = UDim2.new(1, -16, 1, -8)
    btn.Position = UDim2.new(0, 8, 0, 4)
    btn.BackgroundColor3 = Color3.fromRGB(30, 80, 160)
    btn.Text = "🔧  TESTAR CHAT"
    btn.TextSize = 12
    btn.Font = Enum.Font.GothamBold
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BorderSizePixel = 0
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    btn.MouseButton1Click:Connect(function()
        enviarChat("TESTE !")
    end)
end

-- ── Status ────────────────────────────────────────────────────
local StatCard = Card(34)
local StatLbl = Instance.new("TextLabel", StatCard)
StatLbl.Size = UDim2.new(1, 0, 1, 0)
StatLbl.BackgroundTransparency = 1
StatLbl.Text = "⏹  Parado"
StatLbl.TextSize = 12
StatLbl.Font = Enum.Font.GothamSemibold
StatLbl.TextColor3 = Color3.fromRGB(140, 140, 160)
StatLbl.TextXAlignment = Enum.TextXAlignment.Center

-- ── Botão Play/Stop ───────────────────────────────────────────
local PlayCard = Card(42)
local PlayBtn = Instance.new("TextButton", PlayCard)
PlayBtn.Size = UDim2.new(1, -16, 1, -10)
PlayBtn.Position = UDim2.new(0, 8, 0, 5)
PlayBtn.BackgroundColor3 = Color3.fromRGB(20, 160, 65)
PlayBtn.Text = "▶   INICIAR JJs"
PlayBtn.TextSize = 14
PlayBtn.Font = Enum.Font.GothamBold
PlayBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
PlayBtn.BorderSizePixel = 0
Instance.new("UICorner", PlayBtn).CornerRadius = UDim.new(0, 8)

local playing = false

local function setPlay(v)
    playing = v
    if v then
        PlayBtn.Text = "⏹   PARAR JJs"
        PlayBtn.BackgroundColor3 = Color3.fromRGB(180, 30, 30)
        StatLbl.TextColor3 = Color3.fromRGB(60, 220, 120)
    else
        PlayBtn.Text = "▶   INICIAR JJs"
        PlayBtn.BackgroundColor3 = Color3.fromRGB(20, 160, 65)
        StatLbl.Text = "⏹  Parado"
        StatLbl.TextColor3 = Color3.fromRGB(140, 140, 160)
    end
end

PlayBtn.MouseButton1Click:Connect(function()
    if playing then
        parar()
        setPlay(false)
    else
        setPlay(true)
        StatLbl.Text = "▶  Iniciando..."
        iniciar(function(n)
            if n then
                StatLbl.Text = "▶  " .. extenso(n) .. " !  [" .. n .. "/" .. Config.NumFim .. "]"
            else
                setPlay(false)
            end
        end)
    end
end)

-- Rainbow
local hue = 0
RunService.Heartbeat:Connect(function(dt)
    if Config.Rainbow then
        hue = (hue + dt * 0.25) % 1
        local col = Color3.fromHSV(hue, 0.8, 1)
        WinStroke.Color = col
        Top.BackgroundColor3 = Color3.fromHSV(hue, 0.7, 0.45)
    else
        WinStroke.Color = Color3.fromRGB(0, 200, 80)
        Top.BackgroundColor3 = Color3.fromRGB(15, 130, 55)
    end
end)

-- HOME para abrir/fechar
UserInputService.InputBegan:Connect(function(inp, gp)
    if gp then return end
    if inp.KeyCode == Enum.KeyCode.Home then
        Win.Visible = not Win.Visible
    end
end)

-- Notificação inicial
local N = Instance.new("Frame", SG)
N.Size = UDim2.new(0, 290, 0, 48)
N.Position = UDim2.new(1, -300, 1, -62)
N.BackgroundColor3 = Color3.fromRGB(15, 130, 55)
N.BorderSizePixel = 0
Instance.new("UICorner", N).CornerRadius = UDim.new(0, 10)

local NL = Instance.new("TextLabel", N)
NL.Size = UDim2.new(1, -12, 1, 0)
NL.Position = UDim2.new(0, 6, 0, 0)
NL.BackgroundTransparency = 1
NL.Text = "🪖 EB JJs Xeno carregado! Tecla: HOME"
NL.TextSize = 11
NL.Font = Enum.Font.GothamSemibold
NL.TextColor3 = Color3.fromRGB(255, 255, 255)
NL.TextWrapped = true

task.delay(4, function()
    TweenService:Create(N, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Position = UDim2.new(1, 10, 1, -62)}):Play()
    task.wait(0.5)
    N:Destroy()
end)

print("[EB JJs Xeno] Carregado! Formato: UM !, DOIS !, TRES ! ...")
