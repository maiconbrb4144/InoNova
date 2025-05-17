-- Proteção leve e anti-deteção
if getgenv and getgenv()._ajusteAreaLoaded then return end
if getgenv then getgenv()._ajusteAreaLoaded = true end

-- Variáveis obfuscadas
local S = {P = game:GetService("Players"), U = game:GetService("UserInputService"), R = game:GetService("RunService")}
local lP = S.P.LocalPlayer
local cM = workspace.CurrentCamera

-- Proteção da interface
local g = Instance.new("ScreenGui")
g.Name = "UI_" .. tostring(math.random(100000, 999999))
g.ResetOnSpawn = false
pcall(function() g.Parent = game:GetService("CoreGui") end)

-- Monitoramento para tentar evitar detection de UI
g.DescendantAdded:Connect(function(child)
    if child:IsA("LocalScript") or child:IsA("ModuleScript") then
        child:Destroy()
    end
end)

-- Checagem anti-spy básica
task.spawn(function()
    while true do
        task.wait(5)
        for _, v in pairs(g:GetDescendants()) do
            if v:IsA("LocalScript") or v:IsA("ModuleScript") then
                v:Destroy()
            end
        end
    end
end)

-- Painel
local f = Instance.new("Frame")
f.Size = UDim2.new(0, 180, 0, 110)
f.Position = UDim2.new(0.5, -90, 0.5, -55)
f.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
f.BackgroundTransparency = 0.2
f.BorderSizePixel = 0
f.Active = true
f.Draggable = true
f.Parent = g
Instance.new("UICorner", f).CornerRadius = UDim.new(0, 10)

local t = Instance.new("TextLabel", f)
t.Size = UDim2.new(1, 0, 0, 18)
t.BackgroundTransparency = 1
t.Text = "Ajuste de Área"
t.Font = Enum.Font.GothamBold
t.TextSize = 13
t.TextColor3 = Color3.new(1, 1, 1)

local function criarBtn(txt, y, cor)
    local b = Instance.new("TextButton", f)
    b.Size = UDim2.new(0.9, 0, 0, 20)
    b.Position = UDim2.new(0.05, 0, 0, y)
    b.Text = txt
    b.BackgroundColor3 = cor
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.Gotham
    b.TextSize = 12
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)
    return b
end

local tg, sl, aim = criarBtn("Ativar", 20, Color3.fromRGB(200, 60, 60)), criarBtn("Tam: 10", 42, Color3.fromRGB(70, 130, 255)), criarBtn("Mira: OFF", 64, Color3.fromRGB(100, 100, 255))
local on, mira = false, false
local size = 10

local function getHRP(p)
    return p and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
end

local function aplicar()
    for _, p in ipairs(S.P:GetPlayers()) do
        if p ~= lP then
            local h = getHRP(p)
            if h then
                local a = h:FindFirstChild("Alvo") or Instance.new("Part")
                a.Name = "Alvo"
                a.Anchored, a.CanCollide, a.Transparency = true, false, 1
                a.Size = Vector3.new(1, 1, 1)
                a.Position = h.Position + Vector3.new(0, size / 2, 0)
                a.Parent = h
                h.Size = Vector3.new(size, size, size)
                h.CanCollide = false
                h.Transparency = 1
            end
        end
    end
end

local function limpar()
    for _, p in ipairs(S.P:GetPlayers()) do
        local h = getHRP(p)
        if h then
            local a = h:FindFirstChild("Alvo")
            if a then a:Destroy() end
            h.Size = Vector3.new(2, 2, 1)
            h.Transparency = 1
        end
    end
end

local function miraHit()
    if not mira then return end
    local r = workspace:Raycast(cM.CFrame.Position, cM.CFrame.LookVector * 1000, RaycastParams.new())
    if r and r.Instance and r.Instance.Name == "Alvo" then
        local m = r.Instance:FindFirstAncestorOfClass("Model")
        if m and m:FindFirstChild("Humanoid") then
            m.Humanoid:TakeDamage(0.1)
        end
    end
end

S.R.Heartbeat:Connect(function()
    if on then aplicar() end
end)

tg.MouseButton1Click:Connect(function()
    on = not on
    tg.Text = on and "Desativar" or "Ativar"
    tg.BackgroundColor3 = on and Color3.fromRGB(60, 200, 60) or Color3.fromRGB(200, 60, 60)
    if not on then limpar() end
end)

sl.MouseButton1Click:Connect(function()
    size += 2
    if size > 20 then size = 6 end
    sl.Text = "Tam: " .. size
end)

aim.MouseButton1Click:Connect(function()
    mira = not mira
    aim.Text = "Mira: " .. (mira and "ON" or "OFF")
    aim.BackgroundColor3 = mira and Color3.fromRGB(60, 200, 255) or Color3.fromRGB(100, 100, 255)
end)

S.U.InputBegan:Connect(function(i, gpe)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        miraHit()
    end
end)
