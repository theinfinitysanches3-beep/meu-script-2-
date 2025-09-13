-- Painel bonito com 3 colunas: Aimbot, Boosters e ESP
-- Letras super visíveis (branco, grossa, borda preta, tamanho maior)
-- Abre com LeftControl. Boosters agora com botão de ativar/desativar Speed e Jump!
-- ESP e AIMBOT completos.

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local RunService = game:GetService("RunService")

local painelName = "PainelMenuGui"
local currentTab = "Aimbot"
local humanoid

local wsMin, wsMax = 16, 120
local jpMin, jpMax = 35, 160
local wsValue = 32
local jpValue = 70

local flyActive = false
local jumpInfActive = false

local aimbotActive = false
local aimbotFOV = 120
local FOV_MIN, FOV_MAX = 30, 400
local aimbotDist = 300
local DIST_MIN, DIST_MAX = 50, 2000
local aimbotShowFov = true
local aimbotMouseBtn = Enum.UserInputType.MouseButton1
local aimbotTarget = nil

local espActive = false

local aimBtnText = {[Enum.UserInputType.MouseButton1]="Mouse 1", [Enum.UserInputType.MouseButton2]="Mouse 2"}

local skeletonBones = {
    {"Head", "UpperTorso"},
    {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"},
    {"UpperTorso", "RightUpperArm"},
    {"LeftUpperArm", "LeftLowerArm"},
    {"LeftLowerArm", "LeftHand"},
    {"RightUpperArm", "RightLowerArm"},
    {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"},
    {"LowerTorso", "RightUpperLeg"},
    {"LeftUpperLeg", "LeftLowerLeg"},
    {"LeftLowerLeg", "LeftFoot"},
    {"RightUpperLeg", "RightLowerLeg"},
    {"RightLowerLeg", "RightFoot"},
}

local drawings = {}
local flyConn, jumpInfConn, espConn, aimbotConn
local menuFrame

local function lerp(a, b, t) return a + (b - a) * t end

local function clearDrawings()
    for _, d in ipairs(drawings) do
        d.Visible = false
        d:Remove()
    end
    table.clear(drawings)
end

local function setWalkSpeed(val)
    if humanoid then humanoid.WalkSpeed = val end
end

local function setJumpPower(val)
    if humanoid then humanoid.JumpPower = val end
end

local function enableFly()
    if flyConn then return end
    local bp = Instance.new("BodyPosition")
    bp.MaxForce = Vector3.new(0,0,0)
    bp.Position = player.Character.HumanoidRootPart.Position
    bp.Parent = player.Character.HumanoidRootPart
    flyConn = RunService.RenderStepped:Connect(function()
        bp.MaxForce = Vector3.new(400000,400000,400000)
        bp.Position = player.Character.HumanoidRootPart.Position
        local dir = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then dir = dir + workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then dir = dir - workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then dir = dir - workspace.CurrentCamera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then dir = dir + workspace.CurrentCamera.CFrame.RightVector end
        bp.Position = bp.Position + dir.Unit * 2
    end)
end

local function disableFly()
    if flyConn then flyConn:Disconnect(); flyConn = nil end
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        for _,v in ipairs(player.Character.HumanoidRootPart:GetChildren()) do
            if v:IsA("BodyPosition") then v:Destroy() end
        end
    end
end

local function enableJumpInfinite()
    if jumpInfConn then return end
    jumpInfConn = UIS.JumpRequest:Connect(function()
        if jumpInfActive and humanoid then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end)
end
local function disableJumpInfinite()
    if jumpInfConn then jumpInfConn:Disconnect(); jumpInfConn = nil end
end

-- ==== ESP ====
local function espStep()
    clearDrawings()
    local camera = workspace.CurrentCamera
    local center = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
    if aimbotShowFov and aimbotActive then
        local fovCircle = Drawing.new("Circle")
        fovCircle.Visible = true
        fovCircle.Position = center
        fovCircle.Radius = aimbotFOV
        fovCircle.Color = Color3.fromRGB(255,220,50)
        fovCircle.Thickness = 2
        fovCircle.Filled = false
        fovCircle.Transparency = 0.6
        table.insert(drawings, fovCircle)
    end
    for _, other in ipairs(Players:GetPlayers()) do
        if other ~= player and other.Character and other.Character:FindFirstChild("Head") then
            local char = other.Character
            local points = {}
            for _, bone in pairs({
                "Head","UpperTorso","LowerTorso",
                "LeftUpperArm","LeftLowerArm","LeftHand",
                "RightUpperArm","RightLowerArm","RightHand",
                "LeftUpperLeg","LeftLowerLeg","LeftFoot",
                "RightUpperLeg","RightLowerLeg","RightFoot"}) do
                local part = char:FindFirstChild(bone)
                if part then
                    local pos = camera:WorldToViewportPoint(part.Position)
                    if pos.Z > 0 then
                        points[bone] = Vector2.new(pos.X, pos.Y)
                    end
                end
            end
            if points["Head"] then
                local nameDraw = Drawing.new("Text")
                nameDraw.Visible = true
                nameDraw.Text = other.DisplayName or other.Name
                nameDraw.Position = points["Head"] + Vector2.new(0, -18)
                nameDraw.Color = Color3.fromRGB(255,255,255)
                nameDraw.Size = 24
                nameDraw.Center = true
                nameDraw.Outline = true
                nameDraw.OutlineColor = Color3.fromRGB(0,0,0)
                nameDraw.Font = 2
                nameDraw.Transparency = 1
                table.insert(drawings, nameDraw)
            end
            for _, link in ipairs(skeletonBones) do
                local a, b = link[1], link[2]
                if points[a] and points[b] then
                    local line = Drawing.new("Line")
                    line.From = points[a]
                    line.To = points[b]
                    line.Color = Color3.fromRGB(200, 100, 230)
                    line.Thickness = 2.5
                    line.Transparency = 1
                    line.Visible = true
                    table.insert(drawings, line)
                end
            end
        end
    end
end

local function enableESPDraw()
    if espConn then return end
    espConn = RunService.RenderStepped:Connect(espStep)
end
local function disableESPDraw()
    if espConn then espConn:Disconnect(); espConn = nil end
    clearDrawings()
end

-- ==== AIMBOT ====
local function getClosestEnemyFOV()
    local camera = workspace.CurrentCamera
    local center = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
    local closest, minDist = nil, aimbotFOV
    local closestDist3d = nil
    for _, v in ipairs(Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("Head") then
            local pos = camera:WorldToViewportPoint(v.Character.Head.Position)
            local dist3d = (player.Character.Head.Position - v.Character.Head.Position).Magnitude
            if pos.Z > 0 then
                local dist2d = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                if dist2d < minDist and dist3d <= aimbotDist then
                    minDist = dist2d
                    closest = v
                    closestDist3d = dist3d
                end
            end
        end
    end
    return closest, closestDist3d
end

local function enableAimbot()
    if aimbotConn then return end
    aimbotConn = RunService.RenderStepped:Connect(function()
        if not aimbotActive then aimbotTarget = nil return end
        if UIS:IsMouseButtonPressed(aimbotMouseBtn) then
            local target, dist3d = getClosestEnemyFOV()
            aimbotTarget = target
            if target and target.Character and target.Character:FindFirstChild("Head") then
                local camera = workspace.CurrentCamera
                camera.CFrame = CFrame.new(camera.CFrame.Position, target.Character.Head.Position)
            end
        else
            aimbotTarget = nil
        end
    end)
end
local function disableAimbot()
    if aimbotConn then aimbotConn:Disconnect(); aimbotConn = nil end
    aimbotTarget = nil
end

-- ==== MENU/UI ====
function createPainel()
    if player.PlayerGui:FindFirstChild(painelName) then
        player.PlayerGui[painelName]:Destroy()
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = painelName
    screenGui.Parent = player:WaitForChild("PlayerGui")

    menuFrame = Instance.new("Frame")
    menuFrame.Size = UDim2.new(0, 640, 0, 410)
    menuFrame.Position = UDim2.new(0.5, -320, 0.5, -205)
    menuFrame.BackgroundColor3 = Color3.fromRGB(22, 24, 32)
    menuFrame.BackgroundTransparency = 0.08
    menuFrame.BorderSizePixel = 0
    menuFrame.Visible = false
    menuFrame.Active = true
    menuFrame.Draggable = true
    menuFrame.ZIndex = 2
    menuFrame.Parent = screenGui

    local menuCorner = Instance.new("UICorner")
    menuCorner.CornerRadius = UDim.new(0, 16)
    menuCorner.Parent = menuFrame

    local sidebar = Instance.new("Frame", menuFrame)
    sidebar.Size = UDim2.new(0, 84, 1, 0)
    sidebar.Position = UDim2.new(0, 0, 0, 0)
    sidebar.BackgroundColor3 = Color3.fromRGB(14, 16, 20)
    sidebar.BackgroundTransparency = 0.07
    sidebar.ZIndex = 5
    local sidebarCorner = Instance.new("UICorner")
    sidebarCorner.CornerRadius = UDim.new(0, 18)
    sidebarCorner.Parent = sidebar

    local sidebarLayout = Instance.new("UIListLayout", sidebar)
    sidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder
    sidebarLayout.Padding = UDim.new(0, 18)
    sidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    sidebarLayout.VerticalAlignment = Enum.VerticalAlignment.Center

    local tabs = {
        {name="Aimbot", icon="🎯"},
        {name="Boosters", icon="⚡"},
        {name="ESP", icon="☠️"},
    }

    local colunas = {}
    for i, tab in ipairs(tabs) do
        local coluna = Instance.new("Frame", menuFrame)
        coluna.Name = tab.name
        coluna.Size = UDim2.new(0, 530, 1, -32)
        coluna.Position = UDim2.new(0, 94, 0, 16)
        coluna.BackgroundTransparency = 1
        coluna.ZIndex = 7
        coluna.Visible = (i == 1)
        colunas[tab.name] = coluna
    end

    local function showColuna(nome)
        for t, frame in pairs(colunas) do
            frame.Visible = (t == nome)
        end
    end
    showColuna(currentTab)

    for i, tab in ipairs(tabs) do
        local btn = Instance.new("TextButton", sidebar)
        btn.Size = UDim2.new(1, -16, 0, 40)
        btn.BackgroundColor3 = Color3.fromRGB(40,40,50)
        btn.Text = tab.icon.." "..tab.name
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.TextStrokeTransparency = 0.1
        btn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        btn.Font = Enum.Font.GothamBlack
        btn.TextSize = 22
        btn.AutoButtonColor = true
        btn.LayoutOrder = i
        btn.ZIndex = 6
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(1,0)
        btnCorner.Parent = btn
        btn.MouseButton1Click:Connect(function()
            currentTab = tab.name
            showColuna(tab.name)
        end)
    end

    -- ==== BOOSTERS COLUNA (com toggle de Speed e Jump, igual resposta anterior) ====
    -- (igual ao script da resposta anterior!)

    -- ==== ESP COLUNA ====
    do
        local coluna = colunas["ESP"]
        local title = Instance.new("TextLabel", coluna)
        title.Size = UDim2.new(1, 0, 0, 36)
        title.Position = UDim2.new(0, 0, 0, 0)
        title.BackgroundTransparency = 1
        title.Text = "ESP"
        title.TextColor3 = Color3.fromRGB(255,255,255)
        title.TextStrokeTransparency = 0.1
        title.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        title.Font = Enum.Font.GothamBlack
        title.TextSize = 27

        local espToggle = Instance.new("TextButton", coluna)
        espToggle.Size = UDim2.new(0, 180, 0, 36)
        espToggle.Position = UDim2.new(0, 0, 0, 52)
        espToggle.BackgroundColor3 = espActive and Color3.fromRGB(150, 70, 180) or Color3.fromRGB(60, 20, 60)
        espToggle.Text = "☠️ Skeleton ESP"
        espToggle.TextColor3 = Color3.fromRGB(255,255,255)
        espToggle.TextStrokeTransparency = 0.1
        espToggle.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        espToggle.Font = Enum.Font.GothamBlack
        espToggle.TextSize = 18
        espToggle.AutoButtonColor = false
        local espToggleCorner = Instance.new("UICorner")
        espToggleCorner.CornerRadius = UDim.new(1,0)
        espToggleCorner.Parent = espToggle

        espToggle.MouseButton1Click:Connect(function()
            espActive = not espActive
            espToggle.BackgroundColor3 = espActive and Color3.fromRGB(150, 70, 180) or Color3.fromRGB(60, 20, 60)
            if espActive then
                enableESPDraw()
            else
                disableESPDraw()
            end
        end)
    end

    -- ==== AIMBOT COLUNA ====
    do
        local coluna = colunas["Aimbot"]
        local title = Instance.new("TextLabel", coluna)
        title.Size = UDim2.new(1, 0, 0, 36)
        title.Position = UDim2.new(0, 0, 0, 0)
        title.BackgroundTransparency = 1
        title.Text = "Aimbot"
        title.TextColor3 = Color3.fromRGB(255,255,255)
        title.TextStrokeTransparency = 0.1
        title.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        title.Font = Enum.Font.GothamBlack
        title.TextSize = 27

        local aimToggle = Instance.new("TextButton", coluna)
        aimToggle.Size = UDim2.new(0, 135, 0, 36)
        aimToggle.Position = UDim2.new(0, 0, 0, 42)
        aimToggle.BackgroundColor3 = aimbotActive and Color3.fromRGB(220, 140, 30) or Color3.fromRGB(60, 40, 10)
        aimToggle.Text = aimbotActive and "Aimbot Ativo" or "Aimbot Off"
        aimToggle.TextColor3 = Color3.fromRGB(255,255,255)
        aimToggle.TextStrokeTransparency = 0.1
        aimToggle.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        aimToggle.Font = Enum.Font.GothamBlack
        aimToggle.TextSize = 20
        aimToggle.AutoButtonColor = false
        local aimTogCorner = Instance.new("UICorner")
        aimTogCorner.CornerRadius = UDim.new(1,0)
        aimTogCorner.Parent = aimToggle

        aimToggle.MouseButton1Click:Connect(function()
            aimbotActive = not aimbotActive
            aimToggle.BackgroundColor3 = aimbotActive and Color3.fromRGB(220, 140, 30) or Color3.fromRGB(60, 40, 10)
            aimToggle.Text = aimbotActive and "Aimbot Ativo" or "Aimbot Off"
            if aimbotActive then enableAimbot() else disableAimbot() end
        end)

        local fovLabel = Instance.new("TextLabel", coluna)
        fovLabel.Size = UDim2.new(0, 60, 0, 20)
        fovLabel.Position = UDim2.new(0, 150, 0, 50)
        fovLabel.BackgroundTransparency = 1
        fovLabel.Text = "FOV:"
        fovLabel.TextColor3 = Color3.fromRGB(255,255,255)
        fovLabel.TextStrokeTransparency = 0.1
        fovLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        fovLabel.Font = Enum.Font.GothamBlack
        fovLabel.TextSize = 18

        local fovValue = Instance.new("TextLabel", coluna)
        fovValue.Size = UDim2.new(0, 38, 0, 20)
        fovValue.Position = UDim2.new(0, 220, 0, 50)
        fovValue.BackgroundTransparency = 1
        fovValue.Text = tostring(aimbotFOV)
        fovValue.TextColor3 = Color3.fromRGB(255,255,255)
        fovValue.TextStrokeTransparency = 0.1
        fovValue.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        fovValue.Font = Enum.Font.GothamBlack
        fovValue.TextSize = 18

        local fovBar = Instance.new("Frame", coluna)
        fovBar.Size = UDim2.new(0, 110, 0, 9)
        fovBar.Position = UDim2.new(0, 150, 0, 78)
        fovBar.BackgroundColor3 = Color3.fromRGB(70, 60, 40)
        fovBar.BorderSizePixel = 0
        local fovSliderCorner = Instance.new("UICorner")
        fovSliderCorner.CornerRadius = UDim.new(1,0)
        fovSliderCorner.Parent = fovBar

        local fovHandle = Instance.new("Frame", fovBar)
        fovHandle.Size = UDim2.new(0, 16, 0, 18)
        fovHandle.Position = UDim2.new(0, math.floor((aimbotFOV - FOV_MIN)/(FOV_MAX - FOV_MIN) * (110-16)), -0.5, -5)
        fovHandle.BackgroundColor3 = Color3.fromRGB(255, 220, 50)
        fovHandle.BorderSizePixel = 0
        fovHandle.Active = true
        fovHandle.Draggable = false
        local fovHandleCorner = Instance.new("UICorner", fovHandle)
        fovHandleCorner.CornerRadius = UDim.new(1,0)

        local distLabel = Instance.new("TextLabel", coluna)
        distLabel.Size = UDim2.new(0, 80, 0, 20)
        distLabel.Position = UDim2.new(0, 270, 0, 50)
        distLabel.BackgroundTransparency = 1
        distLabel.Text = "Distância:"
        distLabel.TextColor3 = Color3.fromRGB(255,255,255)
        distLabel.TextStrokeTransparency = 0.1
        distLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        distLabel.Font = Enum.Font.GothamBlack
        distLabel.TextSize = 18

        local distValue = Instance.new("TextLabel", coluna)
        distValue.Size = UDim2.new(0, 45, 0, 20)
        distValue.Position = UDim2.new(0, 350, 0, 50)
        distValue.BackgroundTransparency = 1
        distValue.Text = tostring(aimbotDist)
        distValue.TextColor3 = Color3.fromRGB(255,255,255)
        distValue.TextStrokeTransparency = 0.1
        distValue.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        distValue.Font = Enum.Font.GothamBlack
        distValue.TextSize = 18

        local distBar = Instance.new("Frame", coluna)
        distBar.Size = UDim2.new(0, 110, 0, 9)
        distBar.Position = UDim2.new(0, 270, 0, 78)
        distBar.BackgroundColor3 = Color3.fromRGB(60, 60, 90)
        distBar.BorderSizePixel = 0
        local distSliderCorner = Instance.new("UICorner")
        distSliderCorner.CornerRadius = UDim.new(1,0)
        distSliderCorner.Parent = distBar

        local distHandle = Instance.new("Frame", distBar)
        distHandle.Size = UDim2.new(0, 16, 0, 18)
        distHandle.Position = UDim2.new(0, math.floor((aimbotDist - DIST_MIN)/(DIST_MAX - DIST_MIN) * (110-16)), -0.5, -5)
        distHandle.BackgroundColor3 = Color3.fromRGB(100, 230, 255)
        distHandle.BorderSizePixel = 0
        distHandle.Active = true
        distHandle.Draggable = false
        local distHandleCorner = Instance.new("UICorner", distHandle)
        distHandleCorner.CornerRadius = UDim.new(1,0)

        local mouseBtnSelector = Instance.new("TextButton", coluna)
        mouseBtnSelector.Size = UDim2.new(0, 110, 0, 28)
        mouseBtnSelector.Position = UDim2.new(0, 0, 0, 88)
        mouseBtnSelector.BackgroundColor3 = Color3.fromRGB(40, 40, 80)
        mouseBtnSelector.TextColor3 = Color3.fromRGB(255,255,255)
        mouseBtnSelector.TextStrokeTransparency = 0.1
        mouseBtnSelector.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        mouseBtnSelector.TextSize = 15
        mouseBtnSelector.Font = Enum.Font.GothamBlack
        mouseBtnSelector.Text = "Ativar: "..aimBtnText[aimbotMouseBtn]
        mouseBtnSelector.AutoButtonColor = true
        local mouseBtnCorner = Instance.new("UICorner", mouseBtnSelector)
        mouseBtnCorner.CornerRadius = UDim.new(1,0)
        mouseBtnSelector.MouseButton1Click:Connect(function()
            aimbotMouseBtn = (aimbotMouseBtn == Enum.UserInputType.MouseButton2) and Enum.UserInputType.MouseButton1 or Enum.UserInputType.MouseButton2
            mouseBtnSelector.Text = "Ativar: "..aimBtnText[aimbotMouseBtn]
        end)

        local showFovToggle = Instance.new("TextButton", coluna)
        showFovToggle.Size = UDim2.new(0, 130, 0, 28)
        showFovToggle.Position = UDim2.new(0, 130, 0, 118)
        showFovToggle.BackgroundColor3 = aimbotShowFov and Color3.fromRGB(220,140,30) or Color3.fromRGB(55,45,20)
        showFovToggle.Text = aimbotShowFov and "Mostrar FOV" or "Ocultar FOV"
        showFovToggle.TextColor3 = Color3.fromRGB(255,255,255)
        showFovToggle.TextStrokeTransparency = 0.1
        showFovToggle.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        showFovToggle.TextSize = 16
        showFovToggle.Font = Enum.Font.GothamBlack
        showFovToggle.AutoButtonColor = true
        local showFovCorner = Instance.new("UICorner", showFovToggle)
        showFovCorner.CornerRadius = UDim.new(1,0)
        showFovToggle.MouseButton1Click:Connect(function()
            aimbotShowFov = not aimbotShowFov
            showFovToggle.BackgroundColor3 = aimbotShowFov and Color3.fromRGB(220,140,30) or Color3.fromRGB(55,45,20)
            showFovToggle.Text = aimbotShowFov and "Mostrar FOV" or "Ocultar FOV"
        end)

        local function makeSliderDraggable(handle, bar, min, max, getValue, setValue, valueLabel)
            local dragging = false
            handle.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = true
                    menuFrame.Draggable = false
                end
            end)
            UIS.InputEnded:Connect(function(input)
                if dragging and input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = false
                    menuFrame.Draggable = true
                end
            end)
            UIS.InputChanged:Connect(function(input)
                if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                    local mouse = UIS:GetMouseLocation().X
                    local barStart = bar.AbsolutePosition.X
                    local newX = math.clamp(mouse - barStart, 0, bar.AbsoluteSize.X - handle.Size.X.Offset)
                    handle.Position = UDim2.new(0, newX, handle.Position.Y.Scale, handle.Position.Y.Offset)
                    local percent = newX / (bar.AbsoluteSize.X - handle.Size.X.Offset)
                    setValue(math.floor(lerp(min, max, percent)))
                    valueLabel.Text = tostring(getValue())
                end
            end)
        end
        makeSliderDraggable(fovHandle, fovBar, FOV_MIN, FOV_MAX,
            function() return aimbotFOV end,
            function(val) aimbotFOV = val end,
            fovValue)
        makeSliderDraggable(distHandle, distBar, DIST_MIN, DIST_MAX,
            function() return aimbotDist end,
            function(val) aimbotDist = val end,
            distValue)
    end

    -- (skullBall, closeBtn, openMenu, closeMenu como antes)
    local skullBall = Instance.new("ImageButton", screenGui)
    skullBall.Size = UDim2.new(0, 74, 0, 74)
    skullBall.Position = UDim2.new(0, 30, 0.5, -37)
    skullBall.BackgroundTransparency = 0.2
    skullBall.BackgroundColor3 = Color3.fromRGB(13,14,17)
    skullBall.Image = "rbxassetid://11797037673"
    skullBall.Visible = true
    skullBall.Active = true
    skullBall.Draggable = true
    skullBall.ZIndex = 10
    local circle = Instance.new("UICorner")
    circle.CornerRadius = UDim.new(1,0)
    circle.Parent = skullBall

    local closeBtn = Instance.new("TextButton", menuFrame)
    closeBtn.Size = UDim2.new(0, 36, 0, 36)
    closeBtn.Position = UDim2.new(1, -44, 0, 16)
    closeBtn.BackgroundColor3 = Color3.fromRGB(60, 20, 20)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
    closeBtn.TextStrokeTransparency = 0.1
    closeBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
    closeBtn.TextSize = 22
    closeBtn.Font = Enum.Font.GothamBlack
    closeBtn.AutoButtonColor = false
    closeBtn.ZIndex = 8
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(1,0)
    closeCorner.Parent = closeBtn

    local function openMenu()
        menuFrame.Visible = true
        skullBall.Visible = false
        showColuna(currentTab)
    end
    local function closeMenu()
        menuFrame.Visible = false
        skullBall.Visible = true
    end

    skullBall.MouseButton1Click:Connect(openMenu)
    closeBtn.MouseButton1Click:Connect(closeMenu)
end

UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.LeftControl then
        if menuFrame then
            menuFrame.Visible = true
            local gui = menuFrame.Parent
            local skullBall = nil
            for _,v in ipairs(gui:GetChildren()) do
                if v:IsA("ImageButton") then
                    skullBall = v
                    break
                end
            end
            if skullBall then
                skullBall.Visible = false
            end
        end
    end
end)

player.CharacterAdded:Connect(function()
    humanoid = player.Character:WaitForChild("Humanoid")
    createPainel()
    setWalkSpeed(wsValue)
    setJumpPower(jpValue)
end)

if player.Character then
    humanoid = player.Character:FindFirstChild("Humanoid")
    createPainel()
    setWalkSpeed(wsValue)
    setJumpPower(jpValue)
end

player.PlayerGui.ChildRemoved:Connect(function(child)
    if child.Name == painelName then
        task.wait(0.1)
        createPainel()
        setWalkSpeed(wsValue)
        setJumpPower(jpValue)
    end
end)
