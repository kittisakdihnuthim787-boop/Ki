-- โหลด Fluent UI Library
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

-- สร้างหน้าต่าง UI
local Window = Fluent:CreateWindow({
    Title = "Script Hub | by รันดี",
    SubTitle = "The Strongest Battlegrounds",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 420),
    Acrylic = true,
    Theme = "Dark"
})

-- สร้าง Tabs
local Tabs = {
    Main = Window:AddTab({ Title = "ระบบหลัก", Icon = "home" }),
    Settings = Window:AddTab({ Title = "ตั้งค่า UI", Icon = "settings" })
}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

--------------------------------------------------------------------------------
-- สร้างปุ่มย่อ/เปิด UI แบบวงกลม (ลากขยับได้)
--------------------------------------------------------------------------------
local ToggleScreenGui = Instance.new("ScreenGui")
ToggleScreenGui.Name = "RunDeeToggleButton"
ToggleScreenGui.ResetOnSpawn = false

-- ตรวจสอบการรันผ่าน CoreGui หรือ PlayerGui
if syn and syn.protect_gui then
    syn.protect_gui(ToggleScreenGui)
    ToggleScreenGui.Parent = CoreGui
elseif gethui then
    ToggleScreenGui.Parent = gethui()
else
    ToggleScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Name = "CircleButton"
ToggleBtn.Parent = ToggleScreenGui
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Position = UDim2.new(0.05, 0, 0.2, 0) -- ตำแหน่งเริ่มต้น
ToggleBtn.Size = UDim2.new(0, 50, 0, 50) -- ขนาดวงกลม 50x50
ToggleBtn.Font = Enum.Font.SourceSansBold
ToggleBtn.Text = "รันดี"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 16

-- ทำมุมให้เป็นวงกลมและใส่ขอบสี
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(1, 0)
UICorner.Parent = ToggleBtn

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(0, 170, 255)
UIStroke.Thickness = 2
UIStroke.Parent = ToggleBtn

-- ระบบลากปุ่มวงกลมขยับได้บนหน้าจอ
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    ToggleBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

ToggleBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = ToggleBtn.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

ToggleBtn.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- กดปุ่มวงกลมเพื่อ ย่อ/ขยาย UI
ToggleBtn.MouseButton1Click:Connect(function()
    VirtualInputManager = game:GetService("VirtualInputManager")
    -- สลับเปิดปิด UI ของ Fluent
    if Fluent.Window then
        Window:Minimize()
    end
end)

--------------------------------------------------------------------------------
-- ระบบที่ 1: บินตามหลังคนที่อยู่ใกล้ที่สุด (ระยะต่อยถึง)
--------------------------------------------------------------------------------
Tabs.Main:AddParagraph({
    Title = "1. ระบบติดตาม",
    Content = "บินตามหลังคนที่อยู่ใกล้ที่สุดในระยะต่อย (เปิด/ปิด ได้)"
})

local AutoFollowToggle = Tabs.Main:AddToggle("AutoFollow", { Title = "เปิด/ปิด บินตามหลังคนใกล้สุด", Default = false })
local FollowConnection = nil

AutoFollowToggle:OnChanged(function()
    if AutoFollowToggle.Value then
        FollowConnection = RunService.Heartbeat:Connect(function()
            local closestPlayer = nil
            local shortestDistance = math.huge
            local char = LocalPlayer.Character
            
            if not char or not char:FindFirstChild("HumanoidRootPart") then return end
            
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local distance = (char.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
                end
            end
            
            if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local targetHRP = closestPlayer.Character.HumanoidRootPart
                local behindCFrame = targetHRP.CFrame * CFrame.new(0, 0, 2.2)
                char.HumanoidRootPart.CFrame = behindCFrame
            end
        end)
        
        Fluent:Notify({
            Title = "ระบบติดตาม",
            Content = "เริ่มบินตามหลังเป้าหมายแล้ว!",
            Duration = 3
        })
    else
        if FollowConnection then
            FollowConnection:Disconnect()
            FollowConnection = nil
        end
        
        Fluent:Notify({
            Title = "ระบบติดตาม",
            Content = "ปิดการบินตามหลังแล้ว",
            Duration = 3
        })
    end
end)

--------------------------------------------------------------------------------
-- ระบบที่ 2: ดูสมาชิกทั้งหมดในเซิฟ (อัปเดตเรียลไทม์เมื่อเข้า-ออก)
--------------------------------------------------------------------------------
Tabs.Main:AddSection("2. รายชื่อสมาชิกในเซิร์ฟเวอร์")

local PlayerParagraph = Tabs.Main:AddParagraph({
    Title = "รายชื่อผู้เล่นทั้งหมด",
    Content = "กำลังโหลดข้อมูล..."
})

local function UpdatePlayerList()
    local playerListText = ""
    local allPlayers = Players:GetPlayers()
    
    for i, p in ipairs(allPlayers) do
        playerListText = playerListText .. i .. ". " .. p.Name .. " (@" .. p.DisplayName .. ")\n"
    end
    
    PlayerParagraph:SetDesc(playerListText)
end

Players.PlayerAdded:Connect(UpdatePlayerList)
Players.PlayerRemoving:Connect(UpdatePlayerList)
UpdatePlayerList()

--------------------------------------------------------------------------------
-- ระบบที่ 3: กดสกิลทั้งหมด (สนามรบที่แข็งแกร่ง)
--------------------------------------------------------------------------------
Tabs.Main:AddSection("3. ระบบต่อสู้")

Tabs.Main:AddButton({
    Title = "กดสกิลทั้งหมด (1, 2, 3, 4)",
    Callback = function()
        local VirtualInputManager = game:GetService("VirtualInputManager")
        local keysToPress = {Enum.KeyCode.One, Enum.KeyCode.Two, Enum.KeyCode.Three, Enum.KeyCode.Four}
        
        for _, keyCode in ipairs(keysToPress) do
            VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
            task.wait(0.05)
            VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
            task.wait(0.1)
        end
        
        Fluent:Notify({
            Title = "สกิล",
            Content = "ทำการใช้สกิลทั้งหมดเรียบร้อย!",
            Duration = 3
        })
    end
})

--------------------------------------------------------------------------------
-- ตั้งค่าระบบ UI
--------------------------------------------------------------------------------
InterfaceManager:SetLibrary(Fluent)
InterfaceManager:SetFolder("RunDeeScript")
InterfaceManager:BuildInterfaceSection(Tabs.Settings)

Window:SelectTab(1)

Fluent:Notify({
    Title = "Script Hub by รันดี",
    Content = "สร้างปุ่มวงกลม 'รันดี' สำหรับย่อ/เปิด UI บนหน้าจอเรียบร้อย!",
    Duration = 5
})
