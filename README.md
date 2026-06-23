local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LP = Players.LocalPlayer

local settings = {
    SilentAim = true,
    TeamCheck = true,
    WallCheck = true,
    FOV = 150,
    TargetPart = "Head"
}

-- Visualização do FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = settings.FOV
fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Visible = true

local function getClosest()
    local closest, dist = nil, math.huge
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LP and v.Character and v.Character:FindFirstChild(settings.TargetPart) then
            if settings.TeamCheck and v.Team == LP.Team then continue end
            
            local pos, onScreen = Camera:WorldToViewportPoint(v.Character[settings.TargetPart].Position)
            local mag = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
            
            if onScreen and mag < settings.FOV then
                if settings.WallCheck then
                    local ray = workspace:Raycast(Camera.CFrame.Position, (v.Character[settings.TargetPart].Position - Camera.CFrame.Position), RaycastParams.new())
                    if ray and ray.Instance:IsDescendantOf(v.Character) then
                        if mag < dist then closest = v.Character[settings.TargetPart]; dist = mag end
                    end
                else
                    if mag < dist then closest = v.Character[settings.TargetPart]; dist = mag end
                end
            end
        end
    end
    return closest
end

-- Hook de Silent Aim (Intercepta disparos)
local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
    local args = {...}
    local method = getnamecallmethod()
    
    if settings.SilentAim and (method == "FireServer" or method == "InvokeServer") and tostring(self) == "ShootEvent" then
        local target = getClosest()
        if target then
            args[1] = target.Position -- Redireciona o vetor do disparo
            return old(self, unpack(args))
        end
    end
    return old(self, ...)
end)
setreadonly(mt, true)
