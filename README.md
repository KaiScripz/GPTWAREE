# GPTWAREE
Aimlock to its finest


getgenv().OldAimPart = "LowerTorso"
getgenv().AimPart = "LowerTorso"
getgenv().AimlockKey = "c"
getgenv().AimRadius = 30
getgenv().ThirdPerson = true
getgenv().FirstPerson = true
getgenv().TeamCheck = false
getgenv().PredictMovement = true
getgenv().PredictionVelocity = 6.6456  s2
getgenv().CheckIfJumped = true
getgenv().Smoothness = false
getgenv().SmoothnessAmount = 0.5
local Players, Uis, RService, SGui = game:GetService("Players"), game:GetService("UserInputService"), game:GetService("RunService"), game:GetService("StarterGui")
local Client, Mouse, Camera, CF, RNew, Vec3, Vec2 = Players.LocalPlayer, Players.LocalPlayer:GetMouse(), workspace.CurrentCamera, CFrame.new, Ray.new, Vector3.new, Vector2.new
local Aimlock, MousePressed, CanNotify = true, false, false
local AimlockTarget
local OldPre

getgenv().WorldToViewportPoint = function(P)
    return Camera:WorldToViewportPoint(P)
end

getgenv().WorldToScreenPoint = function(P)
    return Camera.WorldToScreenPoint(Camera, P)
end

getgenv().GetObscuringObjects = function(T)
    if T and T:FindFirstChild(getgenv().AimPart) and Client and Client.Character:FindFirstChild("Head") then
        local RayPos = workspace:FindPartOnRay(RNew(
            T[getgenv().AimPart].Position, Client.Character.Head.Position
        ))
        if RayPos then
            return RayPos:IsDescendantOf(T)
        end
    end
end

getgenv().GetNearestTarget = function()
    local players = {}
    local PLAYER_HOLD = {}
    local DISTANCES = {}

    for i, v in pairs(Players:GetPlayers()) do
        if v ~= Client then
            table.insert(players, v)
        end
    end

    for i, v in pairs(players) do
        if v.Character ~= nil then
            local AIM = v.Character:FindFirstChild("Head")

            if getgenv().TeamCheck == true and v.Team ~= Client.Team then
                local DISTANCE = (v.Character:FindFirstChild("Head").Position - game.Workspace.CurrentCamera.CFrame.p).magnitude
                local RAY = Ray.new(game.Workspace.CurrentCamera.CFrame.p, (Mouse.Hit.p - game.Workspace.CurrentCamera.CFrame.p).unit * DISTANCE)
                local HIT, POS = game.Workspace:FindPartOnRay(RAY, game.Workspace)
                local DIFF = math.floor((POS - AIM.Position).magnitude)

                PLAYER_HOLD[v.Name .. i] = {}
                PLAYER_HOLD[v.Name .. i].dist = DISTANCE
                PLAYER_HOLD[v.Name .. i].plr = v
                PLAYER_HOLD[v.Name .. i].diff = DIFF
                table.insert(DISTANCES, DIFF)
            elseif getgenv().TeamCheck == false and v.Team == Client.Team then
                local DISTANCE = (v.Character:FindFirstChild("Head").Position - game.Workspace.CurrentCamera.CFrame.p).magnitude
                local RAY = Ray.new(game.Workspace.CurrentCamera.CFrame.p, (Mouse.Hit.p - game.Workspace.CurrentCamera.CFrame.p).unit * DISTANCE)
                local HIT, POS = game.Workspace:FindPartOnRay(RAY, game.Workspace)
                local DIFF = math.floor((POS - AIM.Position).magnitude)

                PLAYER_HOLD[v.Name .. i] = {}
                PLAYER_HOLD[v.Name .. i].dist = DISTANCE
                PLAYER_HOLD[v.Name .. i].plr = v
                PLAYER_HOLD[v.Name .. i].diff = DIFF
                table.insert(DISTANCES, DIFF)
            end
        end
    end

    if unpack(DISTANCES) == nil then
        return nil
    end

    local TARGET = unpack(DISTANCES)
    local PLR = PLAYER_HOLD[TARGET]

    if PLR and PLR.plr and PLR.plr.Character ~= nil and PLR.plr.Character:FindFirstChild("Head") then
        if PLR.plr.Character:FindFirstChild("Head") and PLR.plr.Character:FindFirstChild(getgenv().AimPart) then
            if getgenv().GetObscuringObjects(PLR.plr.Character) == nil or getgenv().GetObscuringObjects(PLR.plr.Character) == false then
                if AimlockTarget ~= nil and AimlockTarget.Character ~= nil and AimlockTarget:FindFirstChild("Head") then
                    AimlockTarget = nil
                end
                AimlockTarget = PLR.plr.Character
            else
                if AimlockTarget ~= nil and AimlockTarget.Character ~= nil and AimlockTarget:FindFirstChild("Head") then
                    AimlockTarget = nil
                end
            end
        end
    end
end

SGui:SetCore("SendNotification", {
    Title = "Aimlock",
    Text = "Press " .. getgenv().AimlockKey .. " to toggle aimlock",
    Duration = 5
})

RService.RenderStepped:Connect(function()
    if Aimlock and AimlockTarget and AimlockTarget:FindFirstChild("Head") and AimlockTarget:FindFirstChild(getgenv().AimPart) then
        local HeadPos, OnScreen = Camera:WorldToViewportPoint(AimlockTarget.Head.Position)
        local AimPos = Vec2(Mouse.X, Mouse.Y)
        local Distance = (AimPos - Vec2(HeadPos.X, HeadPos.Y)).magnitude

        if OnScreen and Distance <= getgenv().AimRadius then
            local AimAt = Camera:WorldToScreenPoint(AimlockTarget[getgenv().AimPart].Position)
            mousemoverel((AimAt.X - Mouse.X), (AimAt.Y - Mouse.Y))
        else
            AimlockTarget = nil
        end
    end
end)

Uis.InputBegan:Connect(function(Input)
    if Input.UserInputType == Enum.UserInputType.Keyboard then
        local Key = Input.KeyCode

        if Key == Enum.KeyCode[getgenv().AimlockKey] then
            MousePressed = true
        end
    end
end)

Uis.InputEnded:Connect(function(Input)
    if Input.UserInputType == Enum.UserInputType.Keyboard then
        local Key = Input.KeyCode

        if Key == Enum.KeyCode[getgenv().AimlockKey] then
            MousePressed = false
        end
    end
end)

while true do
    if MousePressed then
        Aimlock = not Aimlock
        CanNotify = not CanNotify

        if CanNotify then
            SGui:SetCore("SendNotification", {
                Title = "Aimlock",
                Text = "Aimlock " .. tostring(Aimlock and "enabled" or "disabled"),
                Duration = 5
            })
        end

        wait(0.2)
    end
    getgenv().GetNearestTarget()
    wait()
end
