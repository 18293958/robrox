-- 客户端脚本：UI + 特效 + 聊天播报 + R15 换皮（最终版）
local RS = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local SG = game:GetService("StarterGui")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local KillerSpawn = RS:WaitForChild("KillerSpawn", 10)

-- 黄色小黄人模板
local YellowTemplate = RS:WaitForChild("YellowR15Template", 10)
if not YellowTemplate then
    warn("[变身] ❌ 找不到 YellowR15Template，请放入 ReplicatedStorage！")
end

local monsters = {
    {name = "Boxy Boo",    args = {"BoxyBoo"}, color = Color3.fromRGB(255,100,100)},
    {name = "Huggy Wuggy", args = {"Huggy"},   color = Color3.fromRGB(100,150,255)},
    {name = "妈咪",         args = {"Mommy"},   color = Color3.fromRGB(180,120,255)},
    {name = "Stalky",      args = {"Stalky"},  color = Color3.fromRGB(80,200,120)},
    {name = "是果仁",       args = {"是果仁"},   color = Color3.fromRGB(210,160,90)},
    {name = "🥷《黑客》🥷", args = {"黑客"},    color = Color3.fromRGB(60,60,80)},
}

-- R15 15个关节名称（根据你源码里的命名习惯，加上 Head）
local R15Parts = {
    "Head", "UpperTorso", "LowerTorso",
    "LeftUpperArm", "LeftLowerArm", "LeftHand",
    "RightUpperArm", "RightLowerArm", "RightHand",
    "LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
    "RightUpperLeg", "RightLowerLeg", "RightFoot"
}

-- ========== 核心：客户端换皮函数 ==========
local function applyStalkySkin()
    local char = player.Character
    if not char then return end
    if not YellowTemplate then return end

    for _, partName in ipairs(R15Parts) do
        local targetPart = char:FindFirstChild(partName)
        local templatePart = YellowTemplate:FindFirstChild(partName)
        if targetPart and templatePart and templatePart:IsA("BasePart") then
            -- 替换大小、颜色、材质
            targetPart.Size = templatePart.Size
            targetPart.Color = templatePart.Color
            targetPart.Material = templatePart.Material
            
            -- 替换 Mesh
            local tMesh = targetPart:FindFirstChildOfClass("SpecialMesh")
            local sMesh = templatePart:FindFirstChildOfClass("SpecialMesh")
            if tMesh and sMesh then
                tMesh.MeshId = sMesh.MeshId
                tMesh.TextureId = sMesh.TextureId
                tMesh.Scale = sMesh.Scale
            end
            
            -- 替换贴图
            local tTex = targetPart:FindFirstChildOfClass("Texture")
            local sTex = templatePart:FindFirstChildOfClass("Texture")
            if tTex and sTex then
                tTex.Texture = sTex.Texture
            end
            
            -- 替换 SurfaceAppearance
            local tSA = targetPart:FindFirstChildOfClass("SurfaceAppearance")
            local sSA = templatePart:FindFirstChildOfClass("SurfaceAppearance")
            if tSA and sSA then
                tSA.ColorMap = sSA.ColorMap
                tSA.NormalMap = sSA.NormalMap
                tSA.RoughnessMap = sSA.RoughnessMap
                tSA.MetalnessMap = sSA.MetalnessMap
            end
        end
    end
    print("✅ 客户端换皮完成，已变成小黄人外观！")
end-- ========== 彩虹发光特效 ==========
local redConn, redLoop, hl
local function stopFx()
    if redConn then redConn:Disconnect() redConn = nil end
    if redLoop then redLoop:Disconnect() redLoop = nil end
    if hl then hl:Destroy() hl = nil end
end

local function startFx()
    stopFx()
    local char = player.Character
    if not char then return end
    for _, v in ipairs(char:GetDescendants()) do
        if v:IsA("Decal") or v:IsA("Texture") then v.Transparency = 1
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled = false
        elseif v:IsA("BillboardGui") or v:IsA("SurfaceGui") then v.Enabled = false end
    end
    redConn = char.DescendantAdded:Connect(function(v)
        task.wait(0.1)
        if v:IsA("Decal") or v:IsA("Texture") then v.Transparency = 1
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled = false
        elseif v:IsA("BillboardGui") or v:IsA("SurfaceGui") then v.Enabled = false end
    end)
    hl = Instance.new("Highlight")
    hl.FillTransparency = 0.5
    hl.OutlineTransparency = 0
    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Adornee = char
    hl.Parent = char
    local t = 0
    redLoop = RunService.RenderStepped:Connect(function(dt)
        if not redConn or not player.Character then
            if redLoop then redLoop:Disconnect() end
            return
        end
        t = (t + dt * 0.4) % 1
        local parts = {}
        for _, v in ipairs(player.Character:GetDescendants()) do
            if v:IsA("BasePart") then table.insert(parts, v) end
        end
        local n = math.max(#parts, 1)
        for i, p in ipairs(parts) do
            p.Color = Color3.fromHSV((t + (i-1)/n) % 1, 1, 1)
            p.Material = Enum.Material.SmoothPlastic
        end
        if hl and hl.Parent then
            hl.FillColor = Color3.fromHSV(t, 1, 1)
            hl.OutlineColor = Color3.fromHSV((t + 0.5) % 1, 1, 1)
        end
    end)
end

-- ========== 聊天播报 ==========
local KW = {"松散了","登场了","是果仁","变身","怪物","黑客"}
local function colorize(s)
    if type(s) ~= "string" then return s end
    for _, k in ipairs(KW) do
        if s:find(k, 1, true) then
            local out = ""
            for i = 1, #s do
                local c = Color3.fromHSV(((i-1)/#s) % 1, 1, 1)
                out = out .. string.format('<font color="rgb(%d,%d,%d)">%s</font>',
                    c.R*255, c.G*255, c.B*255, s:sub(i,i))
            end
            return out
        end
    end
    return s
end
local hooked = {}
local function hook(r)
    if not r:IsA("RemoteEvent") or hooked[r] then return end
    hooked[r] = true
    r.OnClientEvent:Connect(function(...)
        for _, v in ipairs({...}) do
            if type(v) == "string" then
                local c = colorize(v)
                if c ~= v then
                    pcall(function()
                        SG:SetCore("ChatMakeSystemMessage",
                            {Text = c, Color = Color3.new(1,1,1), Font = Enum.Font.GothamBold})
                    end)
                end
            end
        end
    end)
end
for _, o in ipairs(RS:GetDescendants()) do hook(o) end
RS.DescendantAdded:Connect(hook)

-- ========== UI ==========
local gui = Instance.new("ScreenGui")
gui.Name = "MonsterSelectUI"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local open = Instance.new("TextButton")
open.Size = UDim2.new(0, 42, 0, 42)
open.Position = UDim2.new(0, 16, 0.5, -21)
open.BackgroundColor3 = Color3.fromRGB(120, 80, 255)
open.Text = "🎭"
open.TextScaled = true
open.Font = Enum.Font.GothamBold
open.Draggable = true
open.Parent = gui
Instance.new("UICorner", open).CornerRadius = UDim.new(1, 0)

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 330)
frame.Position = UDim2.new(0.5, -100, 0.5, -165)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
frame.BorderSizePixel = 0
frame.Draggable = true
frame.Visible = false
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
local st = Instance.new("UIStroke", frame)
st.Color = Color3.fromRGB(120, 120, 255)
st.Thickness = 2

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -30, 0, 26)
title.Position = UDim2.new(0, 8, 0, 4)
title.BackgroundTransparency = 1
title.Text = "🎭 选择怪物变身"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = frame

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -16, 0, 18)
status.Position = UDim2.new(0, 8, 1, -22)
status.BackgroundTransparency = 1
status.Text = "点击按钮进行变身"
status.TextColor3 = Color3.fromRGB(180, 180, 180)
status.TextScaled = true
status.Font = Enum.Font.GothamBold
status.RichText = true
status.Parent = frame

local statusConn
local function startStatusRainbow()
    if statusConn then statusConn:Disconnect() end
    local t = 0
    statusConn = RunService.Heartbeat:Connect(function(dt)
        t = (t + dt * 0.6) % 1
        status.TextColor3 = Color3.fromHSV(t, 1, 1)
    end)
end
local function stopStatusRainbow()
    if statusConn then statusConn:Disconnect() statusConn = nil end
end

local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 18, 0, 18)
close.Position = UDim2.new(1, -23, 0, 6)
close.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
close.Text = "X"
close.TextColor3 = Color3.new(1, 1, 1)
close.TextScaled = true
close.Font = Enum.Font.GothamBold
close.Parent = frame
Instance.new("UICorner", close).CornerRadius = UDim.new(0, 4)

for i, m in ipairs(monsters) do
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, -24, 0, 32)
    b.Position = UDim2.new(0, 12, 0, 34 + (i - 1) * 36)
    b.BackgroundColor3 = m.color
    b.Text = m.name
    b.TextColor3 = Color3.new(1, 1, 1)
    b.TextScaled = true
    b.Font = Enum.Font.GothamBold
    b.Parent = frame
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)

    b.MouseButton1Click:Connect(function()
        stopStatusRainbow()
        status.TextColor3 = Color3.fromRGB(255, 220, 100)
        status.Text = "正在变身: " .. m.name .. "..."
        
        -- 发送服务器请求（处理游戏逻辑）
        local ok = pcall(function()
            KillerSpawn:FireServer(table.unpack(m.args))
        end)
        
        -- 如果是 Stalky，直接在本地换皮 + 启动特效
        if m.name == "Stalky" then
            task.wait(0.1)
            applyStalkySkin()
            startFx()
        else
            stopFx()
        end
        
        if ok then
            status.Text = "✅ 已请求变身: " .. m.name
            startStatusRainbow()
        else
            status.TextColor3 = Color3.fromRGB(255, 100, 100)
            status.Text = "❌ 发送失败"
        end
    end)
end

open.MouseButton1Click:Connect(function()
    frame.Visible = true
    open.Visible = false
end)
close.MouseButton1Click:Connect(function()
    frame.Visible = false
    open.Visible = true
    stopStatusRainbow()
end)

print("[变身脚本] 已加载，共 " .. #monsters .. " 个怪物")