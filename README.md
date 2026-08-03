--[[
    Henika Hub - Modern Duel Steal / Brainrot Script
    Reconstructed from AdaptHub UI, rebranded and enhanced.
    Removed all "adapt.vs" references, added Bat Aimbot mode selector (Normal / V2),
    and TP Bat for anti-bat counter.
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

--------------------------------------------------------------------------------
-- SHARED STYLE CONSTANTS
--------------------------------------------------------------------------------
local DARK_BG         = Color3.fromRGB(0, 0, 0)
local DARKER_BG       = Color3.fromRGB(5, 5, 8)
local INPUT_BG        = Color3.fromRGB(8, 8, 12)
local WHITE           = Color3.fromRGB(255, 255, 255)
local MUTED_TEXT      = Color3.fromRGB(190, 187, 202)
local TRAIL_COLOR     = Color3.fromRGB(205, 215, 235)

local BORDER_GRAD_DARK = {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 40, 40)),
        ColorSequenceKeypoint.new(0.25, Color3.fromRGB(25, 25, 25)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(40, 40, 40)),
        ColorSequenceKeypoint.new(0.75, Color3.fromRGB(20, 20, 20)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(40, 40, 40)),
    }),
    Rotation = 135,
    Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.92, 0),
        NumberSequenceKeypoint.new(0.25, 0.65, 0),
        NumberSequenceKeypoint.new(0.5, 0.38, 0),
        NumberSequenceKeypoint.new(0.75, 0.72, 0),
        NumberSequenceKeypoint.new(1, 0.92, 0),
    }),
}

local BORDER_GRAD_LIGHT = {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.25, Color3.fromRGB(180, 185, 210)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.75, Color3.fromRGB(160, 165, 200)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 255, 255)),
    }),
    Rotation = 135,
    Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5, 0),
        NumberSequenceKeypoint.new(0.25, 0.2, 0),
        NumberSequenceKeypoint.new(0.5, 0, 0),
        NumberSequenceKeypoint.new(0.75, 0.2, 0),
        NumberSequenceKeypoint.new(1, 0.5, 0),
    }),
}

local ARROW_GLOW_TRANSPARENCY = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0.82, 0),
    NumberSequenceKeypoint.new(0.28, 0.06, 0),
    NumberSequenceKeypoint.new(0.52, 0.22, 0),
    NumberSequenceKeypoint.new(1, 0.82, 0),
})

local BUTTON_NONE_GRADIENT_COLOR = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(22, 22, 25)),
    ColorSequenceKeypoint.new(0.18, Color3.fromRGB(2, 2, 3)),
    ColorSequenceKeypoint.new(0.82, Color3.fromRGB(2, 2, 3)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(22, 22, 25)),
})

--------------------------------------------------------------------------------
-- HELPERS
--------------------------------------------------------------------------------
local function applyProps(inst, props)
    for k, v in pairs(props) do
        if k ~= "Parent" then
            inst[k] = v
        end
    end
    if props.Parent then
        inst.Parent = props.Parent
    end
    return inst
end

local function new(class, props)
    return applyProps(Instance.new(class), props)
end

local function addDarkBorderGradient(stroke)
    local g = Instance.new("UIGradient")
    g.Color = BORDER_GRAD_DARK.Color
    g.Rotation = BORDER_GRAD_DARK.Rotation
    g.Transparency = BORDER_GRAD_DARK.Transparency
    g.Parent = stroke
    return g
end

local function addLightBorderGradient(stroke)
    local g = Instance.new("UIGradient")
    g.Color = BORDER_GRAD_LIGHT.Color
    g.Rotation = BORDER_GRAD_LIGHT.Rotation
    g.Transparency = BORDER_GRAD_LIGHT.Transparency
    g.Parent = stroke
    return g
end

local function addRowBorder(parent, thickness)
    new("UICorner", {CornerRadius = UDim.new(0, 9), Parent = parent})
    local stroke = new("UIStroke", {
        Color = WHITE,
        Thickness = thickness or 1.25,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
        Parent = parent,
    })
    addDarkBorderGradient(stroke)
    return stroke
end

local function addInputBorder(parent, cornerRadius)
    new("UICorner", {CornerRadius = UDim.new(0, cornerRadius or 7), Parent = parent})
    local stroke = new("UIStroke", {
        Color = WHITE,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
        Parent = parent,
    })
    addDarkBorderGradient(stroke)
    return stroke
end

local function addRowLabel(parent, text, fontSize)
    return new("TextLabel", {
        ZIndex = 4,
        Position = UDim2.new(0, 12, 0, 0),
        Size = UDim2.new(1, -132, 1, 0),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = WHITE,
        TextSize = fontSize or 13,
        Font = Enum.Font.GothamMedium,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = parent,
    })
end

local function createRow(name, parent)
    return new("Frame", {
        Name = name,
        ZIndex = 3,
        Size = UDim2.new(1, -4, 0, 34),
        BackgroundColor3 = DARK_BG,
        BackgroundTransparency = 0.3,
        BorderSizePixel = 0,
        Parent = parent,
    })
end

local function createSection(name, text, parent)
    return new("TextLabel", {
        Name = "Section_" .. name,
        ZIndex = 4,
        Size = UDim2.new(1, -6, 0, 15),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = WHITE,
        TextSize = 11,
        Font = Enum.Font.GothamBlack,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = parent,
    })
end

-- Value Input Row
local function createValueRow(name, label, defaultValue, parent)
    local row = createRow(name, parent)
    addRowBorder(row)
    addRowLabel(row, label)

    local box = new("TextBox", {
        Name = "ValueBox",
        ZIndex = 25,
        Position = UDim2.new(1, -68, 0.5, -12),
        Size = UDim2.new(0, 58, 0, 24),
        BackgroundColor3 = INPUT_BG,
        BackgroundTransparency = 0.18,
        BorderSizePixel = 0,
        Text = tostring(defaultValue),
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        ClearTextOnFocus = false,
        Parent = row,
    })
    addInputBorder(box)
    return row
end

-- Toggle Row
local function createToggle(name, label, parent, hasArrow)
    local row = createRow(name, parent)
    addRowBorder(row)
    addRowLabel(row, label)

    if hasArrow then
        local arrow = new("TextButton", {
            Name = "ArrowButton",
            ZIndex = 5,
            Position = UDim2.new(1, -108, 0.5, -13),
            Size = UDim2.new(0, 38, 0, 26),
            BackgroundColor3 = INPUT_BG,
            BackgroundTransparency = 0.18,
            Text = "▼",
            TextColor3 = WHITE,
            TextSize = 22,
            Font = Enum.Font.GothamBlack,
            AutoButtonColor = false,
            Parent = row,
        })
        new("UICorner", {CornerRadius = UDim.new(0, 7), Parent = arrow})
        local border = new("UIStroke", {Name = "AnimatedArrowBorder", Color = WHITE, Thickness = 1.8, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Transparency = 0.05, Parent = arrow})
        new("UIGradient", {Rotation = 135, Transparency = ARROW_GLOW_TRANSPARENCY, Parent = border})
        local glow = new("UIStroke", {Name = "AnimatedArrowGlow", Color = WHITE, Thickness = 3.6, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Transparency = 0.58, Parent = arrow})
        new("UIGradient", {Name = "GlowGradient", Rotation = 180, Transparency = ARROW_GLOW_TRANSPARENCY, Parent = glow})
    end

    local toggleBtn = new("TextButton", {
        Position = UDim2.new(1, -54, 0, 0),
        Size = UDim2.new(0, 54, 1, 0),
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Parent = row,
    })

    local track = new("Frame", {
        ZIndex = 5,
        Position = UDim2.new(0.5, -17, 0.5, -9),
        Size = UDim2.new(0, 34, 0, 18),
        BackgroundColor3 = WHITE,
        BackgroundTransparency = 0.2,
        BorderSizePixel = 0,
        Parent = toggleBtn,
    })
    new("UICorner", {CornerRadius = UDim.new(0, 9), Parent = track})
    local trackStroke = new("UIStroke", {Color = WHITE, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = track})
    addDarkBorderGradient(trackStroke)

    local knob = new("Frame", {
        ZIndex = 6,
        Position = UDim2.new(1, -16, 0.5, -6),
        Size = UDim2.new(0, 13, 0, 13),
        BackgroundColor3 = DARK_BG,
        BorderSizePixel = 0,
        Parent = track,
    })
    new("UICorner", {CornerRadius = UDim.new(0, 7), Parent = knob})
    new("Frame", {
        ZIndex = 7,
        Position = UDim2.new(0, 2, 0, 2),
        Size = UDim2.new(1, -4, 0, 4),
        BackgroundColor3 = WHITE,
        BackgroundTransparency = 0.72,
        BorderSizePixel = 0,
        Parent = knob,
    })

    new("TextButton", {
        ZIndex = 100,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Parent = toggleBtn,
    })

    return row
end

-- Expandable frame (option selector)
local function createExpandable(option1, option2, parent)
    local frame = createRow("Frame", parent)
    frame.Visible = false
    frame.Size = UDim2.new(1, -4, 0, 0)
    addRowBorder(frame)

    local highlight = new("Frame", {
        ZIndex = 4,
        Position = UDim2.new(0, 4, 0, 4),
        Size = UDim2.new(0.5, -4, 1, -8),
        BackgroundColor3 = WHITE,
        BackgroundTransparency = 0.85,
        BorderSizePixel = 0,
        Parent = frame,
    })
    new("UICorner", {Parent = highlight})
    local hStroke = new("UIStroke", {Color = WHITE, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = highlight})
    addLightBorderGradient(hStroke)

    new("TextButton", {
        ZIndex = 5,
        Size = UDim2.new(0.5, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = option1,
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false,
        Parent = frame,
    })
    new("TextButton", {
        ZIndex = 5,
        Position = UDim2.new(0.5, 0, 0, 0),
        Size = UDim2.new(0.5, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = option2,
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false,
        Parent = frame,
    })

    return frame
end

-- Mode Row (click to cycle)
local function createModeRow(name, label, defaultMode, parent)
    local row = createRow(name, parent)
    addRowBorder(row)
    addRowLabel(row, label, 11)

    new("TextLabel", {
        Name = "ModeValue",
        ZIndex = 6,
        Position = UDim2.new(1, -150, 0, 0),
        Size = UDim2.new(0, 138, 1, 0),
        BackgroundTransparency = 1,
        Text = defaultMode,
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        TextXAlignment = Enum.TextXAlignment.Right,
        Parent = row,
    })
    new("TextButton", {
        Name = "ModeClick",
        ZIndex = 30,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Parent = row,
    })

    return row
end

-- Keybind Row
local function createKeybindRow(name, label, defaultKey, parent)
    local row = createRow(name, parent)
    addRowBorder(row)
    addRowLabel(row, label)

    local btn = new("TextButton", {
        Name = "KeybindButton",
        ZIndex = 5,
        Position = UDim2.new(1, -64, 0.5, -11),
        Size = UDim2.new(0, 56, 0, 22),
        BackgroundColor3 = INPUT_BG,
        BackgroundTransparency = 0.18,
        Text = defaultKey,
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false,
        Parent = row,
    })
    addInputBorder(btn)

    local clearBtn = new("TextButton", {
        Name = "ClearKeybindButton",
        ZIndex = 5,
        Position = UDim2.new(1, -86, 0.5, -9),
        Size = UDim2.new(0, 18, 0, 18),
        BackgroundColor3 = INPUT_BG,
        BackgroundTransparency = 0.08,
        Text = "",
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false,
        Parent = row,
    })
    new("UICorner", {CornerRadius = UDim.new(1, 0), Parent = clearBtn})
    local cStroke = new("UIStroke", {Color = WHITE, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = clearBtn})
    addDarkBorderGradient(cStroke)

    return row
end

-- Action Button Row
local function createActionButton(name, label, parent)
    local row = createRow(name, parent)
    addRowBorder(row)
    addRowLabel(row, label)
    new("TextButton", {
        ZIndex = 200,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Parent = row,
    })
    return row
end

-- Picker Row (< value >)
local function createPickerRow(name, label, defaultValue, parent, prevName, nextName)
    local row = new("Frame", {
        Name = name,
        ZIndex = 3,
        Size = UDim2.new(1, -4, 0, 52),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Parent = parent,
    })
    new("TextLabel", {
        ZIndex = 4,
        Position = UDim2.new(0, 2, 0, 0),
        Size = UDim2.new(1, 0, 0, 16),
        BackgroundTransparency = 1,
        Text = label,
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamBlack,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local prev = new("TextButton", {
        Name = prevName or "TextButton",
        ZIndex = 5,
        Position = UDim2.new(0, 0, 0, 22),
        Size = UDim2.new(0, 44, 0, 25),
        BackgroundColor3 = INPUT_BG,
        BackgroundTransparency = 0.18,
        Text = "<",
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false,
        Parent = row,
    })
    addInputBorder(prev)
    local val = new("TextButton", {
        Name = nextName and (name:gsub(" ", "") .. "Value") or "TextButton",
        ZIndex = 5,
        Position = UDim2.new(0, 48, 0, 22),
        Size = UDim2.new(1, -96, 0, 28),
        BackgroundColor3 = INPUT_BG,
        BackgroundTransparency = 0.18,
        Text = defaultValue,
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false,
        Parent = row,
    })
    addInputBorder(val)
    local nxt = new("TextButton", {
        Name = nextName or "TextButton",
        ZIndex = 5,
        Position = UDim2.new(1, -44, 0, 22),
        Size = UDim2.new(0, 44, 0, 25),
        BackgroundColor3 = INPUT_BG,
        BackgroundTransparency = 0.18,
        Text = ">",
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false,
        Parent = row,
    })
    addInputBorder(nxt)
    return row
end

-- Thumbnail Gallery
local function createThumbnailGallery(name, thumbImages, parent, prefix)
    prefix = prefix or "Bg"
    local frame = new("Frame", {
        Name = name,
        ZIndex = 3,
        Size = UDim2.new(1, -4, 0, 52),
        BackgroundColor3 = DARK_BG,
        BackgroundTransparency = 0.3,
        BorderSizePixel = 0,
        Parent = parent,
    })
    new("UICorner", {CornerRadius = UDim.new(0, 11), Parent = frame})
    local fStroke = new("UIStroke", {Color = WHITE, Thickness = 1.25, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = frame})
    addDarkBorderGradient(fStroke)

    local scroll = new("ScrollingFrame", {
        Name = prefix .. "Scroll",
        ZIndex = 5,
        Size = UDim2.new(1, -56, 1, 0),
        Position = UDim2.new(0.5, 0, 0, 0),
        AnchorPoint = Vector2.new(0.5, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 0,
        ScrollBarImageTransparency = 1,
        ScrollingDirection = Enum.ScrollingDirection.X,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.X,
        Parent = frame,
    })
    new("UIListLayout", {
        FillDirection = Enum.FillDirection.Horizontal,
        Padding = UDim.new(0, 4),
        SortOrder = Enum.SortOrder.LayoutOrder,
        VerticalAlignment = Enum.VerticalAlignment.Center,
        Parent = scroll,
    })
    new("UIPadding", {PaddingLeft = UDim.new(0, 4), PaddingRight = UDim.new(0, 4), Parent = scroll})

    for i, imageId in ipairs(thumbImages) do
        local thumb = new("ImageButton", {
            Name = prefix .. "Thumb" .. i,
            ZIndex = 5,
            LayoutOrder = i,
            Size = UDim2.new(0, 42, 0, 34),
            BackgroundColor3 = DARK_BG,
            BorderSizePixel = 0,
            Image = imageId or "",
            ImageTransparency = 0.22,
            ScaleType = Enum.ScaleType.Crop,
            AutoButtonColor = false,
            Parent = scroll,
        })
        new("UICorner", {CornerRadius = UDim.new(0, 9), Parent = thumb})
        new("UIStroke", {Color = Color3.fromRGB(80, 80, 90), Transparency = 0.4, Parent = thumb})
        if i == 1 then
            new("TextLabel", {
                Name = "NoneLabel",
                ZIndex = 6,
                Size = UDim2.new(1, 0, 1, 0),
                BackgroundTransparency = 1,
                Text = "NONE",
                TextColor3 = WHITE,
                TextSize = 9,
                Font = Enum.Font.GothamMedium,
                Parent = thumb,
            })
        end
    end

    for _, data in ipairs({
        {name = prefix .. "ArrowLeft",  text = "<", pos = UDim2.new(0, 3, 0.5, -17)},
        {name = prefix .. "ArrowRight", text = ">", pos = UDim2.new(1, -25, 0.5, -17)},
    }) do
        local arrow = new("TextButton", {
            Name = data.name,
            ZIndex = 8,
            Position = data.pos,
            Size = UDim2.new(0, 22, 0, 34),
            BackgroundColor3 = Color3.fromRGB(18, 18, 23),
            BackgroundTransparency = 0.05,
            BorderSizePixel = 0,
            Text = data.text,
            TextColor3 = WHITE,
            TextSize = 18,
            Font = Enum.Font.GothamMedium,
            Parent = frame,
        })
        new("UICorner", {CornerRadius = UDim.new(0, 7), Parent = arrow})
    end

    return frame
end

-- Color Theme Picker
local function createColorThemePicker(parent)
    local row = new("Frame", {
        Name = "ColorThemePicker",
        ZIndex = 3,
        Size = UDim2.new(1, -4, 0, 30),
        BackgroundColor3 = DARK_BG,
        BackgroundTransparency = 0.3,
        BorderSizePixel = 0,
        Parent = parent,
    })
    new("UICorner", {CornerRadius = UDim.new(0, 9), Parent = row})
    local rStroke = new("UIStroke", {Color = WHITE, Thickness = 1.25, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = row})
    addDarkBorderGradient(rStroke)

    local colors = {
        {"PURPLE", Color3.fromRGB(207, 159, 255), -141},
        {"BLUE",   Color3.fromRGB(58, 128, 245),  -105},
        {"RED",    Color3.fromRGB(232, 52, 68),    -69},
        {"PINK",   Color3.fromRGB(255, 105, 180),  -33},
        {"YELLOW", Color3.fromRGB(255, 214, 0),    3},
        {"GREY",   Color3.fromRGB(13, 13, 13),     39},
        {"WHITE2", Color3.fromRGB(255, 255, 255),  75},
        {"FOREST", Color3.fromRGB(46, 139, 87),    111},
    }
    for _, data in ipairs(colors) do
        local btn = new("TextButton", {
            Name = data[1] == "WHITE2" and "WHITE" or data[1],
            ZIndex = 5,
            Position = UDim2.new(0.5, data[3], 0.5, -6),
            Size = UDim2.new(0, 30, 0, 12),
            BackgroundColor3 = data[2],
            Text = "",
            AutoButtonColor = false,
            Parent = row,
        })
        new("UICorner", {CornerRadius = UDim.new(0, 4), Parent = btn})
        new("UIStroke", {Color = WHITE, Transparency = 0.5, Parent = btn})
    end
    return row
end

-- Tab Page
local function createTabPage(name, parent, visible)
    local page = new("ScrollingFrame", {
        Name = name,
        Visible = visible ~= false,
        ZIndex = 3,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 0,
        ScrollBarImageTransparency = 1,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = parent,
    })
    new("UIListLayout", {Padding = UDim.new(0, 7), SortOrder = Enum.SortOrder.LayoutOrder, Parent = page})
    return page
end

-- Tab Button
local function createTabButton(name, text, order, isActive, parent)
    local btn = new("TextButton", {
        Name = name,
        ZIndex = 4,
        LayoutOrder = order,
        Size = UDim2.new(0.166667, -5, 1, 0),
        BackgroundColor3 = WHITE,
        BackgroundTransparency = isActive and 0.78 or 1,
        Text = text,
        TextColor3 = isActive and WHITE or MUTED_TEXT,
        TextSize = 11,
        TextScaled = true,
        Font = Enum.Font.GothamBlack,
        TextWrapped = true,
        AutoButtonColor = false,
        Parent = parent,
    })
    new("UITextSizeConstraint", {MinTextSize = 6, MaxTextSize = 12, Parent = btn})
    new("UICorner", {CornerRadius = UDim.new(0, 9), Parent = btn})
    local stroke = new("UIStroke", {Color = WHITE, Thickness = 1.2, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = btn})
    if isActive then
        addLightBorderGradient(stroke)
    else
        addDarkBorderGradient(stroke)
    end
    return btn
end

-- Mobile Button
local function createMobileButton(name, text, position, size, parent)
    local btn = new("TextButton", {
        Name = name,
        ClipsDescendants = true,
        Position = position,
        Size = size,
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Parent = parent,
    })
    new("UICorner", {CornerRadius = UDim.new(0, 10), Parent = btn})
    new("UIScale", {Parent = btn})

    local bg = new("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundColor3 = WHITE,
        Parent = btn,
    })
    new("UICorner", {CornerRadius = UDim.new(0, 10), Parent = bg})
    new("UIGradient", {Name = "ButtonNoneGradient", Color = BUTTON_NONE_GRADIENT_COLOR, Rotation = 25, Parent = bg})

    local overlay = new("ImageLabel", {
        Name = "ImageLabel",
        ZIndex = 2,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Image = "",
        ImageTransparency = 1,
        ScaleType = Enum.ScaleType.Crop,
        Parent = btn,
    })
    new("UICorner", {CornerRadius = UDim.new(0, 10), Parent = overlay})

    local lbl = new("TextLabel", {
        ZIndex = 3,
        Position = UDim2.new(0, 4, 0, 0),
        Size = UDim2.new(1, -8, 1, 0),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = WHITE,
        Font = Enum.Font.GothamBlack,
        TextWrapped = true,
        Parent = btn,
    })
    new("UIStroke", {Thickness = 1.4, Parent = lbl})

    local borderStroke = new("UIStroke", {
        Color = WHITE,
        Thickness = 1.1,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
        Transparency = 0.55,
        Parent = btn,
    })
    addDarkBorderGradient(borderStroke)
    return btn
end

--------------------------------------------------------------------------------
-- ROOT GUI
--------------------------------------------------------------------------------
local HenikaHub = new("ScreenGui", {
    Name = "HenikaHub",
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    Parent = LocalPlayer:WaitForChild("PlayerGui"),
})

--------------------------------------------------------------------------------
-- INTRO SCREEN
--------------------------------------------------------------------------------
local HenikaIntro = new("Frame", {
    Name = "HenikaIntro",
    ZIndex = 1000,
    Size = UDim2.new(1, 0, 1, 0),
    BackgroundColor3 = DARKER_BG,
    BackgroundTransparency = 0.5,
    BorderSizePixel = 0,
    Parent = HenikaHub,
})

-- Intro Banner (rebranded)
new("TextLabel", {
    Name = "IntroBanner",
    ZIndex = 1002,
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.new(0.5, 0, 0.42, 0),
    Size = UDim2.new(0.6, 0, 0, 60),
    BackgroundTransparency = 1,
    Text = "HENIKA HUB",
    TextColor3 = WHITE,
    TextSize = 36,
    Font = Enum.Font.GothamBlack,
    TextScaled = true,
    Parent = HenikaIntro,
})

-- Tap Anywhere
new("TextLabel", {
    Name = "TapAnywhere",
    ZIndex = 1003,
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.new(0.5, 0, 0.42, 60),
    Size = UDim2.new(0.7, 0, 0, 20),
    BackgroundTransparency = 1,
    Text = "TAP ANYWHERE TO SKIP",
    TextColor3 = WHITE,
    TextSize = 11,
    Font = Enum.Font.GothamBlack,
    Parent = HenikaIntro,
})

-- Tap Catcher
local TapCatcher = new("TextButton", {
    Name = "TapCatcher",
    ZIndex = 1004,
    Size = UDim2.new(1, 0, 1, 0),
    BackgroundTransparency = 1,
    Text = "",
    AutoButtonColor = false,
    Parent = HenikaIntro,
})

--------------------------------------------------------------------------------
-- MAIN FRAME
--------------------------------------------------------------------------------
local Main = new("Frame", {
    Name = "Main",
    ClipsDescendants = true,
    AnchorPoint = Vector2.new(0, 0.5),
    Position = UDim2.new(0, 20, 0.5, 0),
    Size = UDim2.new(0, 356, 0, 536),
    BackgroundColor3 = DARK_BG,
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    Parent = HenikaHub,
})
new("UICorner", {CornerRadius = UDim.new(0, 14), Parent = Main})
local mainStroke = new("UIStroke", {Color = WHITE, Thickness = 1.2, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = Main})
addDarkBorderGradient(mainStroke)
new("UIScale", {Scale = 0.85, Parent = Main})

new("ImageLabel", {
    Name = "BackgroundAsset",
    Size = UDim2.new(1, 0, 1, 0),
    BackgroundColor3 = DARK_BG,
    BackgroundTransparency = 1,
    Image = "rbxassetid://90453834580322",
    Parent = Main,
})

new("ImageLabel", {
    Name = "LogoAsset",
    ZIndex = 2,
    Position = UDim2.new(0.5, -150, 0, -18),
    Size = UDim2.new(0, 300, 0, 150),
    BackgroundTransparency = 1,
    Image = "rbxassetid://135088241492683", -- original, can replace
    ScaleType = Enum.ScaleType.Fit,
    Parent = Main,
})

local Close = new("TextButton", {
    Name = "Close",
    ZIndex = 4,
    Position = UDim2.new(1, -46, 0, 9),
    Size = UDim2.new(0, 36, 0, 30),
    BackgroundColor3 = DARK_BG,
    BackgroundTransparency = 0.28,
    Text = "-",
    TextColor3 = WHITE,
    TextSize = 22,
    Font = Enum.Font.GothamMedium,
    AutoButtonColor = false,
    Parent = Main,
})
new("UICorner", {Parent = Close})
local closeStroke = new("UIStroke", {Color = WHITE, ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = Close})
addDarkBorderGradient(closeStroke)

local Content = new("Frame", {
    Name = "Content",
    ZIndex = 3,
    Position = UDim2.new(0, 13, 0, 112),
    Size = UDim2.new(1, -26, 1, -164),
    BackgroundTransparency = 1,
    Parent = Main,
})

--------------------------------------------------------------------------------
-- TABS
--------------------------------------------------------------------------------
local Movement = createTabPage("Movement", Content)
local Combat   = createTabPage("Combat", Content, false)
local Keybinds = createTabPage("Keybinds", Content, false)
local Controller = createTabPage("Controller", Content, false)
local Utility  = createTabPage("Utility", Content, false)
local Settings = createTabPage("Settings", Content, false)

--------------------------------------------------------------------------------
-- MOVEMENT TAB
--------------------------------------------------------------------------------
createSection("Speed Configuration", "SPEED CONFIGURATION", Movement)
createValueRow("Normal Speed", "Normal Speed", "59.5", Movement)
createValueRow("Carry Speed", "Carry Speed", "28.8", Movement)
createModeRow("Speed Mode", "MODE", "NORMAL", Movement)

createSection("Lagger Configuration", "LAGGER CONFIGURATION", Movement)
createValueRow("Lagger Normal Speed", "Lagger Normal Speed", "24.5", Movement)
createValueRow("Lagger Carry Speed", "Lagger Carry Speed", "15", Movement)
createModeRow("Lagger Mode Display", "MODE", "LAGGER NORMAL", Movement)

createSection("Drop Brainrot", "DROP BRAINROT", Movement)
createToggle("Drop", "Drop", Movement, true)
createExpandable("JUMP", "STAND", Movement)

createSection("TP Down", "TP DOWN", Movement)
createToggle("TP Down", "TP Down", Movement, true)
createToggle("Auto TP Down", "Auto TP Down", Movement, true)
createValueRow("Auto TP Height", "Auto TP Height", "20", Movement)

createSection("Jump", "JUMP", Movement)
createToggle("Infinite Jump", "Infinite Jump", Movement, true)
createExpandable("JUMP", "STAND", Movement)
createToggle("Anti Ragdoll", "Anti Ragdoll", Movement, true)
createToggle("Unwalk", "Unwalk", Movement, true)

--------------------------------------------------------------------------------
-- COMBAT TAB (with Bat Aimbot Mode picker)
--------------------------------------------------------------------------------
createSection("Steal Configuration", "STEAL CONFIGURATION", Combat)
createValueRow("Radius", "Radius", "60", Combat)
createValueRow("SEMI Range", "SEMI Range", "0", Combat)
createToggle("Auto Steal", "Auto Steal", Combat, true)
createExpandable("JUMP", "STAND", Combat)

createSection("Bat Aimbot", "BAT AIMBOT", Combat)
-- Add a picker to select aimbot mode (Normal / V2)
createPickerRow("BatAimbotMode", "Aimbot Mode", "NORMAL", Combat, "BatAimbotPrev", "BatAimbotNext")
createToggle("Bat Aimbot", "Bat Aimbot", Combat, true)
createExpandable("JUMP", "STAND", Combat)
createValueRow("Auto Bat Speed", "Auto Bat Speed", "1", Combat)
createToggle("Auto Swing", "Auto Swing", Combat, true)
createToggle("Mirror TP", "Mirror TP", Combat, true)

createSection("TP Bat", "TP BAT", Combat)
createToggle("TP Bat", "TP Bat", Combat, true)
createExpandable("JUMP", "STAND", Combat)

createSection("Auto Path", "AUTO PATH", Combat)
createToggle("Auto Left", "Auto Left", Combat, true)
createToggle("Auto Right", "Auto Right", Combat, true)

createSection("Counters", "COUNTERS", Combat)
createToggle("Bat Counter", "Bat Counter", Combat, true)
createToggle("Medusa Counter", "Medusa Counter", Combat, true)
createToggle("Reset After Med", "Reset After Med", Combat, true)

local ResetAfterMedSettings = createRow("ResetAfterMedSettings", Combat)
ResetAfterMedSettings.Visible = false
ResetAfterMedSettings.Size = UDim2.new(1, -4, 0, 0)
addRowBorder(ResetAfterMedSettings)
createToggle("Insta Reset On Death", "Insta Reset On Death", Combat, true)

createSection("Body Lock", "BODY LOCK", Combat)
createToggle("Body Lock", "Body Lock", Combat, true)
createValueRow("Lock Radius", "Lock Radius", "60", Combat)

--------------------------------------------------------------------------------
-- KEYBINDS TAB (unchanged)
--------------------------------------------------------------------------------
local keybindData = {
    {"MOVEMENT KEYBINDS", {
        {"Speed Key", "Q"},
        {"Lagger Mode Key", "R"},
        {"Drop Key", "X"},
        {"TP Down Key", "F"},
    }},
    {"COMBAT KEYBINDS", {
        {"Bat Aimbot Key", "E"},
        {"TP Bat Key", "V"},
        {"Auto Left Key", "Z"},
        {"Auto Right Key", "C"},
        {"Insta Reset Key", "T"},
    }},
    {"INTERFACE KEYBINDS", {
        {"UI Toggle Key", "N"},
    }},
}
for _, sectionData in ipairs(keybindData) do
    createSection(sectionData[1]:gsub(" ", "_"), sectionData[1], Keybinds)
    for _, bind in ipairs(sectionData[2]) do
        createKeybindRow(bind[1], bind[1], bind[2], Keybinds)
    end
end

--------------------------------------------------------------------------------
-- CONTROLLER TAB (unchanged)
--------------------------------------------------------------------------------
createActionButton("RESET ALL CONTROLLER", "RESET ALL CONTROLLER", Controller)
local controllerData = {
    {"MOVEMENT CONTROLLER", {
        {"Speed Key", "NONE"},
        {"Lagger Mode Key", "NONE"},
        {"Drop Key", "NONE"},
        {"TP Down Key", "NONE"},
    }},
    {"COMBAT CONTROLLER", {
        {"Bat Aimbot Key", "NONE"},
        {"TP Bat Key", "NONE"},
        {"Auto Left Key", "NONE"},
        {"Auto Right Key", "NONE"},
        {"Insta Reset Key", "NONE"},
    }},
    {"INTERFACE CONTROLLER", {
        {"UI Toggle Key", "NONE"},
    }},
}
for _, sectionData in ipairs(controllerData) do
    createSection(sectionData[1]:gsub(" ", "_"), sectionData[1], Controller)
    for _, bind in ipairs(sectionData[2]) do
        createKeybindRow(bind[1], bind[1], bind[2], Controller)
    end
end

--------------------------------------------------------------------------------
-- UTILITY TAB (unchanged)
--------------------------------------------------------------------------------
createSection("Players", "PLAYERS", Utility)
createToggle("ESP", "ESP", Utility, true)
createToggle("Show Tracer", "Show Tracer", Utility, false)
createToggle("Ragdoll Countdown", "Ragdoll Countdown", Utility, false)

createSection("Visual", "VISUAL", Utility)
createPickerRow("Custom Sky", "Custom Sky", "OFF", Utility)
createPickerRow("Anim Pack", "Anim Pack", "OFF", Utility, "AnimPackPrev", "AnimPackNext")
createToggle("Try Hard Animation", "Try Hard Animation", Utility, false)

createSection("Performance", "PERFORMANCE", Utility)
createToggle("Stretch Res", "Stretch Res", Utility, false)
createToggle("Anti Lag", "Anti-Lag", Utility, false)
createToggle("FOV Change", "FOV Change", Utility, false)
createValueRow("FOV Value", "FOV Value", "120", Utility)

--------------------------------------------------------------------------------
-- SETTINGS TAB (unchanged, except removed adapt.vs references)
--------------------------------------------------------------------------------
createSection("Mobile Buttons", "MOBILE BUTTONS", Settings)
createToggle("Circle Buttons", "Circle Buttons", Settings, false)
createToggle("Hide Mob Buttons", "Hide Mob Buttons", Settings, false)

local HideSpecificButtonsList = createRow("HideSpecificButtonsList", Settings)
HideSpecificButtonsList.Visible = false
HideSpecificButtonsList.Size = UDim2.new(1, -4, 0, 0)
addRowBorder(HideSpecificButtonsList)

createValueRow("Button Size %", "Button Size %", "100", Settings)
createToggle("Move Buttons", "Move Buttons", Settings, false)
createActionButton("Reset Buttons", "Reset Buttons", Settings)

createSection("Interface", "INTERFACE", Settings)
createToggle("Intro Song", "Intro Song", Settings, false)
createToggle("Intro", "Intro", Settings, false)

createSection("Background", "BACKGROUND", Settings)
local bgImages = {
    "",
    "rbxassetid://90631990302263",
    "rbxassetid://109619268613730",
    "rbxassetid://88369503310562",
    "rbxassetid://80708025126373",
    "rbxassetid://102253425322931",
    "rbxassetid://90453834580322",
    "rbxassetid://135181794444219",
}
createThumbnailGallery("BackgroundPicker", bgImages, Settings, "Bg")

createSection("Buttons Image", "BUTTONS IMAGE", Settings)
local btnImages = {
    "",
    "rbxassetid://90631990302263",
    "rbxassetid://111941119745474",
    "rbxassetid://88369503310562",
    "rbxassetid://80708025126373",
    "rbxassetid://102253425322931",
    "rbxassetid://138739435956313",
    "rbxassetid://135181794444219",
}
createThumbnailGallery("ButtonsImagePicker", btnImages, Settings, "BtnImg")

createColorThemePicker(Settings)
createToggle("Background Color", "Background Color", Settings, false)

createSection("UI Scale", "UI SCALE", Settings)
createValueRow("UI Scale", "UI Scale", "85", Settings)
createValueRow("Steal Bar Size", "Steal Bar Size", "100", Settings)

createActionButton("SAVE SETTINGS", "SAVE SETTINGS", Settings)
createActionButton("RESET ALL SETTINGS", "RESET ALL SETTINGS", Settings)

--------------------------------------------------------------------------------
-- TABS BAR
--------------------------------------------------------------------------------
local Tabs = new("Frame", {
    Name = "Tabs",
    ZIndex = 3,
    Position = UDim2.new(0, 13, 1, -44),
    Size = UDim2.new(1, -26, 0, 34),
    BackgroundTransparency = 1,
    Parent = Main,
})
new("UIListLayout", {
    Padding = UDim.new(0, 6),
    FillDirection = Enum.FillDirection.Horizontal,
    SortOrder = Enum.SortOrder.LayoutOrder,
    Parent = Tabs,
})

local tabDefs = {
    {"Movement",  "MOVEMENT", 1, true},
    {"Combat",    "COMBAT",   2, false},
    {"Keybinds",  "KBM",      3, false},
    {"Controller","CTRL",     4, false},
    {"Utility",   "VISUAL",   5, false},
    {"Settings",  "SETTINGS", 6, false},
}
for _, def in ipairs(tabDefs) do
    createTabButton(def[1], def[2], def[3], def[4], Tabs)
end

--------------------------------------------------------------------------------
-- FLOATING OPEN BUTTON
--------------------------------------------------------------------------------
local HenikaFloatOpen = new("Frame", {
    Name = "HenikaFloatOpen",
    Visible = false,
    Active = true,
    ZIndex = 500,
    Position = UDim2.new(0, 14, 0.4, 0),
    Size = UDim2.new(0, 110, 0, 32),
    BackgroundColor3 = Color3.fromRGB(14, 14, 18),
    BackgroundTransparency = 0.1,
    BorderSizePixel = 0,
    Parent = HenikaHub,
})
new("UICorner", {Parent = HenikaFloatOpen})
new("UIStroke", {Color = WHITE, Transparency = 0.45, Parent = HenikaFloatOpen})

new("ImageButton", {
    ZIndex = 501,
    Position = UDim2.new(0, 4, 0, 4),
    Size = UDim2.new(1, -8, 1, -8),
    BackgroundTransparency = 1,
    Image = "rbxassetid://92966351305582",
    ScaleType = Enum.ScaleType.Fit,
    AutoButtonColor = false,
    Parent = HenikaFloatOpen,
})

--------------------------------------------------------------------------------
-- MOBILE BUTTONS
--------------------------------------------------------------------------------
local MobileButtons = new("Frame", {
    Name = "MobileButtons",
    Size = UDim2.new(1, 0, 1, 0),
    BackgroundTransparency = 1,
    Parent = HenikaHub,
})
new("UIScale", {Parent = MobileButtons})

local mobileBtnDefs = {
    {"Drop Brainrot",  "DROP BRAINROT",  UDim2.new(1, -132, 0.5, -161), UDim2.new(0, 58, 0, 58)},
    {"Auto Right",     "AUTO RIGHT",     UDim2.new(1, -66,  0.5, -161), UDim2.new(0, 58, 0, 58)},
    {"Bat Aimbot",     "BAT AIMBOT",     UDim2.new(1, -132, 0.5, -95),  UDim2.new(0, 58, 0, 58)},
    {"Auto Left",      "AUTO LEFT",      UDim2.new(1, -66,  0.5, -95),  UDim2.new(0, 58, 0, 58)},
    {"TP Down",        "TP DOWN",        UDim2.new(1, -132, 0.5, -29),  UDim2.new(0, 58, 0, 58)},
    {"Lagger Mode",    "LAGGER MODE",    UDim2.new(1, -66,  0.5, -29),  UDim2.new(0, 58, 0, 58)},
    {"TP Bat",         "TP BAT",         UDim2.new(1, -132, 0.5, 37),   UDim2.new(0, 58, 0, 58)},
    {"Carry Speed",    "CARRY SPEED",    UDim2.new(1, -66,  0.5, 37),   UDim2.new(0, 58, 0, 58)},
    {"Instant Reset",  "INSTANT RESET",  UDim2.new(1, -132, 0.5, 103),  UDim2.new(0, 124, 0, 58)},
}
for _, def in ipairs(mobileBtnDefs) do
    createMobileButton(def[1], def[2], def[3], def[4], MobileButtons)
end

--------------------------------------------------------------------------------
-- STATE MANAGEMENT
--------------------------------------------------------------------------------
local toggleStates = {}   -- [rowFrame] = boolean
local keybindStates = {}  -- [rowFrame] = Enum.KeyCode or nil
local listeningFor = nil
local selectedButtonImage = ""

-- Bat Aimbot mode selection state (stored globally)
local batAimbotMode = "NORMAL"  -- can be "NORMAL" or "V2"

--------------------------------------------------------------------------------
-- HELPERS / TWEEN
--------------------------------------------------------------------------------
local function tween(obj, duration, props, style, dir)
    local info = TweenInfo.new(duration, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out)
    local t = TweenService:Create(obj, info, props)
    t:Play()
    return t
end

local function setToggleVisual(row, state)
    for _, child in ipairs(row:GetChildren()) do
        if child:IsA("TextButton") and child.Name == "TextButton" then
            local track = child:FindFirstChild("Frame")
            if not track then
                for _, c in ipairs(child:GetChildren()) do
                    if c:IsA("Frame") then track = c; break end
                end
            end
            if track then
                local knob
                for _, c in ipairs(track:GetChildren()) do
                    if c:IsA("Frame") then knob = c; break end
                end
                if knob then
                    if state then
                        tween(knob, 0.15, {Position = UDim2.new(1, -16, 0.5, -6), BackgroundColor3 = WHITE})
                        tween(track, 0.15, {BackgroundTransparency = 0})
                    else
                        tween(knob, 0.15, {Position = UDim2.new(0, 3, 0.5, -6), BackgroundColor3 = DARK_BG})
                        tween(track, 0.15, {BackgroundTransparency = 0.2})
                    end
                end
            end
            break
        end
    end
end

local function toggleRow(row)
    local current = toggleStates[row]
    if current == nil then current = false end
    current = not current
    toggleStates[row] = current
    setToggleVisual(row, current)
    return current
end

--------------------------------------------------------------------------------
-- WIRING FUNCTIONS (adapted from original, with changes)
--------------------------------------------------------------------------------
local function wireToggles(page)
    for _, row in ipairs(page:GetChildren()) do
        if row:IsA("Frame") and row.Name ~= "Frame" then
            for _, child in ipairs(row:GetChildren()) do
                if child:IsA("TextButton") and child.Name == "TextButton" then
                    local clickCatcher = child:FindFirstChild("TextButton")
                    if clickCatcher and clickCatcher:IsA("TextButton") then
                        clickCatcher.MouseButton1Click:Connect(function()
                            toggleRow(row)
                        end)
                    else
                        child.MouseButton1Click:Connect(function()
                            toggleRow(row)
                        end)
                    end
                    toggleStates[row] = false
                    setToggleVisual(row, false)
                    break
                end
            end
        end
    end
end

local function wireArrowToggles(page)
    local children = page:GetChildren()
    for i, row in ipairs(children) do
        if row:IsA("Frame") then
            local arrow = row:FindFirstChild("ArrowButton")
            if arrow and arrow:IsA("TextButton") then
                local expandable = nil
                for j = i + 1, #children do
                    local sibling = children[j]
                    if sibling:IsA("Frame") and sibling.Name == "Frame" then
                        expandable = sibling
                        break
                    end
                end
                if expandable then
                    local expanded = false
                    arrow.MouseButton1Click:Connect(function()
                        expanded = not expanded
                        if expanded then
                            expandable.Visible = true
                            tween(expandable, 0.2, {Size = UDim2.new(1, -4, 0, 34)})
                            tween(arrow, 0.15, {Rotation = 180})
                        else
                            tween(arrow, 0.15, {Rotation = 0})
                            tween(expandable, 0.2, {Size = UDim2.new(1, -4, 0, 0)})
                            task.delay(0.2, function()
                                if not expanded then expandable.Visible = false end
                            end)
                        end
                    end)
                end
            end
        end
    end
end

local function wireExpandables(page)
    for _, frame in ipairs(page:GetChildren()) do
        if frame:IsA("Frame") and frame.Name == "Frame" then
            local buttons = {}
            local highlight
            for _, child in ipairs(frame:GetChildren()) do
                if child:IsA("TextButton") then
                    table.insert(buttons, child)
                elseif child:IsA("Frame") and child.BackgroundTransparency < 1 then
                    highlight = child
                end
            end
            if highlight and #buttons >= 2 then
                for idx, btn in ipairs(buttons) do
                    btn.MouseButton1Click:Connect(function()
                        if idx == 1 then
                            tween(highlight, 0.15, {Position = UDim2.new(0, 4, 0, 4)})
                        else
                            tween(highlight, 0.15, {Position = UDim2.new(0.5, 0, 0, 4)})
                        end
                    end)
                end
            end
        end
    end
end

local function wireModeRows(page)
    for _, row in ipairs(page:GetChildren()) do
        if row:IsA("Frame") then
            local modeClick = row:FindFirstChild("ModeClick")
            local modeValue = row:FindFirstChild("ModeValue")
            if modeClick and modeValue then
                local modes
                if row.Name:find("Speed") then
                    modes = {"NORMAL", "LAGGER", "BOTH"}
                elseif row.Name:find("Lagger") then
                    modes = {"LAGGER NORMAL", "LAGGER CARRY", "LAGGER BOTH"}
                else
                    modes = {"DEFAULT", "OPTION 1", "OPTION 2"}
                end
                local modeIdx = 1
                modeClick.MouseButton1Click:Connect(function()
                    modeIdx = modeIdx % #modes + 1
                    modeValue.Text = modes[modeIdx]
                end)
            end
        end
    end
end

local function wireKeybindRows(page)
    for _, row in ipairs(page:GetChildren()) do
        if row:IsA("Frame") then
            local keybindBtn = row:FindFirstChild("KeybindButton")
            local clearBtn = row:FindFirstChild("ClearKeybindButton")
            if keybindBtn and keybindBtn:IsA("TextButton") then
                keybindBtn.MouseButton1Click:Connect(function()
                    if listeningFor == row then
                        listeningFor = nil
                        keybindBtn.Text = keybindStates[row] and keybindStates[row].Name or "NONE"
                        return
                    end
                    listeningFor = row
                    keybindBtn.Text = "..."
                    local conn
                    conn = UserInputService.InputBegan:Connect(function(input, processed)
                        if processed then return end
                        if listeningFor ~= row then
                            conn:Disconnect()
                            return
                        end
                        if input.UserInputType == Enum.UserInputType.Keyboard or input.UserInputType == Enum.UserInputType.Gamepad1 then
                            keybindStates[row] = input.KeyCode
                            keybindBtn.Text = input.KeyCode.Name
                            listeningFor = nil
                            conn:Disconnect()
                        end
                    end)
                end)
            end
            if clearBtn and clearBtn:IsA("TextButton") then
                clearBtn.MouseButton1Click:Connect(function()
                    keybindStates[row] = nil
                    if keybindBtn then keybindBtn.Text = "NONE" end
                    if listeningFor == row then listeningFor = nil end
                end)
            end
        end
    end
end

local function wireValueBoxes(page)
    for _, row in ipairs(page:GetChildren()) do
        if row:IsA("Frame") then
            local box = row:FindFirstChild("ValueBox")
            if box and box:IsA("TextBox") then
                local lastValid = box.Text
                box.FocusLost:Connect(function()
                    local num = tonumber(box.Text)
                    if num then
                        lastValid = tostring(num)
                        box.Text = lastValid
                    else
                        box.Text = lastValid
                    end
                end)
            end
        end
    end
end

-- Picker rows (including the new Bat Aimbot Mode picker)
local function wirePickerRows(page)
    for _, row in ipairs(page:GetChildren()) do
        if row:IsA("Frame") and row.BackgroundTransparency >= 1 then
            local buttons = {}
            local valueBtn = nil
            for _, child in ipairs(row:GetChildren()) do
                if child:IsA("TextButton") then
                    if child.Text == "<" then
                        buttons.prev = child
                    elseif child.Text == ">" then
                        buttons.next = child
                    elseif child.Text ~= "" then
                        valueBtn = child
                    end
                end
            end
            if buttons.prev and buttons.next and valueBtn then
                local options
                if row.Name:find("Custom Sky") then
                    options = {"OFF", "SUNSET", "NIGHT", "GALAXY", "AURORA", "BLOOD"}
                elseif row.Name:find("Anim") then
                    options = {"OFF", "PACK 1", "PACK 2", "PACK 3", "PACK 4"}
                elseif row.Name == "BatAimbotMode" then
                    options = {"NORMAL", "V2"}
                else
                    options = {"OFF", "OPTION 1", "OPTION 2"}
                end
                local idx = 1
                -- Set initial value to match global if applicable
                if row.Name == "BatAimbotMode" then
                    -- find index of current mode
                    for i, opt in ipairs(options) do
                        if opt == batAimbotMode then idx = i; break end
                    end
                    valueBtn.Text = batAimbotMode
                else
                    valueBtn.Text = options[idx]
                end

                local function updatePicker()
                    local val = valueBtn.Text
                    if row.Name == "BatAimbotMode" then
                        batAimbotMode = val
                    end
                end

                buttons.prev.MouseButton1Click:Connect(function()
                    idx = idx - 1
                    if idx < 1 then idx = #options end
                    valueBtn.Text = options[idx]
                    updatePicker()
                    tween(buttons.prev, 0.08, {Size = UDim2.new(0, 40, 0, 23)})
                    task.delay(0.08, function()
                        tween(buttons.prev, 0.08, {Size = UDim2.new(0, 44, 0, 25)})
                    end)
                end)
                buttons.next.MouseButton1Click:Connect(function()
                    idx = idx % #options + 1
                    valueBtn.Text = options[idx]
                    updatePicker()
                    tween(buttons.next, 0.08, {Size = UDim2.new(0, 40, 0, 23)})
                    task.delay(0.08, function()
                        tween(buttons.next, 0.08, {Size = UDim2.new(0, 44, 0, 25)})
                    end)
                end)
            end
        end
    end
end

local function wireThumbnailGalleries(page)
    for _, frame in ipairs(page:GetChildren()) do
        if frame:IsA("Frame") then
            local scroll
            for _, child in ipairs(frame:GetChildren()) do
                if child:IsA("ScrollingFrame") then scroll = child; break end
            end
            if scroll then
                local selectedThumb = nil
                for _, thumb in ipairs(scroll:GetChildren()) do
                    if thumb:IsA("ImageButton") then
                        thumb.MouseButton1Click:Connect(function()
                            if selectedThumb then
                                local oldStroke = selectedThumb:FindFirstChildWhichIsA("UIStroke")
                                if oldStroke then
                                    tween(oldStroke, 0.15, {Color = Color3.fromRGB(80, 80, 90), Transparency = 0.4})
                                end
                                tween(selectedThumb, 0.15, {ImageTransparency = 0.22})
                            end
                            selectedThumb = thumb
                            local stroke = thumb:FindFirstChildWhichIsA("UIStroke")
                            if stroke then tween(stroke, 0.15, {Color = WHITE, Transparency = 0}) end
                            tween(thumb, 0.15, {ImageTransparency = 0})
                            if frame.Name == "BackgroundPicker" then
                                local bgAsset = Main:FindFirstChild("BackgroundAsset")
                                if bgAsset then bgAsset.Image = thumb.Image end
                            end
                            if frame.Name == "ButtonsImagePicker" then
                                selectedButtonImage = thumb.Image or ""
                                updateAllMobileButtonImages()
                            end
                        end)
                    end
                end
                for _, child in ipairs(frame:GetChildren()) do
                    if child:IsA("TextButton") then
                        if child.Text == "<" then
                            child.MouseButton1Click:Connect(function()
                                local pos = scroll.CanvasPosition
                                tween(scroll, 0.2, {CanvasPosition = Vector2.new(math.max(0, pos.X - 50), 0)})
                            end)
                        elseif child.Text == ">" then
                            child.MouseButton1Click:Connect(function()
                                local pos = scroll.CanvasPosition
                                tween(scroll, 0.2, {CanvasPosition = Vector2.new(pos.X + 50, 0)})
                            end)
                        end
                    end
                end
            end
        end
    end
end

local function wireColorThemePicker(page)
    local picker
    for _, child in ipairs(page:GetChildren()) do
        if child.Name == "ColorThemePicker" then picker = child; break end
    end
    if not picker then return end
    local selectedBtn = nil
    for _, btn in ipairs(picker:GetChildren()) do
        if btn:IsA("TextButton") then
            btn.MouseButton1Click:Connect(function()
                if selectedBtn then
                    local oldStroke = selectedBtn:FindFirstChildWhichIsA("UIStroke")
                    if oldStroke then tween(oldStroke, 0.1, {Transparency = 0.5, Thickness = 1}) end
                end
                selectedBtn = btn
                local stroke = btn:FindFirstChildWhichIsA("UIStroke")
                if stroke then tween(stroke, 0.1, {Transparency = 0, Thickness = 2}) end
                local bgAsset = Main:FindFirstChild("BackgroundAsset")
                if bgAsset then tween(bgAsset, 0.3, {ImageColor3 = btn.BackgroundColor3}) end
            end)
        end
    end
end

local function wireActionButtons(page)
    for _, row in ipairs(page:GetChildren()) do
        if row:IsA("Frame") then
            for _, child in ipairs(row:GetChildren()) do
                if child:IsA("TextButton") and child.ZIndex == 200 then
                    child.MouseButton1Click:Connect(function()
                        tween(row, 0.08, {BackgroundTransparency = 0.05})
                        task.delay(0.15, function()
                            tween(row, 0.15, {BackgroundTransparency = 0.3})
                        end)
                        if row.Name == "RESET ALL SETTINGS" or row.Name == "RESET ALL CONTROLLER" then
                            for _, r in ipairs(page:GetChildren()) do
                                if toggleStates[r] ~= nil then
                                    toggleStates[r] = false
                                    setToggleVisual(r, false)
                                end
                            end
                            for _, r in ipairs(page:GetChildren()) do
                                local kb = r:FindFirstChild("KeybindButton")
                                if kb then
                                    keybindStates[r] = nil
                                    kb.Text = "NONE"
                                end
                            end
                        end
                    end)
                    break
                end
            end
        end
    end
end

local function updateAllMobileButtonImages()
    local mbPos = MobileButtons.AbsolutePosition
    local mbSize = MobileButtons.AbsoluteSize
    for _, btn in ipairs(MobileButtons:GetChildren()) do
        if btn:IsA("TextButton") then
            for _, child in ipairs(btn:GetChildren()) do
                if child:IsA("ImageLabel") and child.ZIndex == 2 then
                    child.Image = selectedButtonImage
                    local btnPos = btn.AbsolutePosition
                    child.Size = UDim2.new(0, mbSize.X, 0, mbSize.Y)
                    child.Position = UDim2.new(0, mbPos.X - btnPos.X, 0, mbPos.Y - btnPos.Y)
                    break
                end
            end
        end
    end
end

local function wireMobileButtons()
    local function layoutOverlay(btn, overlay)
        local mbPos = MobileButtons.AbsolutePosition
        local mbSize = MobileButtons.AbsoluteSize
        local btnPos = btn.AbsolutePosition
        if mbSize.X > 0 and mbSize.Y > 0 then
            overlay.Size = UDim2.new(0, mbSize.X, 0, mbSize.Y)
            overlay.Position = UDim2.new(0, mbPos.X - btnPos.X, 0, mbPos.Y - btnPos.Y)
        end
    end
    task.spawn(function()
        task.wait(0.2)
        for _, btn in ipairs(MobileButtons:GetChildren()) do
            if btn:IsA("TextButton") then
                for _, child in ipairs(btn:GetChildren()) do
                    if child:IsA("ImageLabel") and child.ZIndex == 2 then
                        layoutOverlay(btn, child)
                        break
                    end
                end
            end
        end
    end)
    MobileButtons:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
        task.wait()
        for _, btn in ipairs(MobileButtons:GetChildren()) do
            if btn:IsA("TextButton") then
                for _, child in ipairs(btn:GetChildren()) do
                    if child:IsA("ImageLabel") and child.ZIndex == 2 then
                        layoutOverlay(btn, child)
                        break
                    end
                end
            end
        end
    end)

    for _, btn in ipairs(MobileButtons:GetChildren()) do
        if btn:IsA("TextButton") then
            local isActive = false
            local overlay
            local scale = btn:FindFirstChild("UIScale")
            for _, child in ipairs(btn:GetChildren()) do
                if child:IsA("ImageLabel") and child.ZIndex == 2 then overlay = child; break end
            end
            btn.MouseButton1Click:Connect(function()
                if scale then
                    tween(scale, 0.06, {Scale = 0.92})
                    task.delay(0.06, function()
                        tween(scale, 0.1, {Scale = 1})
                    end)
                end
                if overlay then
                    overlay.Image = selectedButtonImage
                    layoutOverlay(btn, overlay)
                end
                if btn.Name == "Instant Reset" then
                    if overlay then
                        tween(overlay, 0.08, {ImageTransparency = 0.15})
                        task.delay(0.25, function()
                            tween(overlay, 0.2, {ImageTransparency = 1})
                        end)
                    end
                    return
                end
                isActive = not isActive
                if overlay then
                    if isActive then tween(overlay, 0.15, {ImageTransparency = 0.15})
                    else tween(overlay, 0.15, {ImageTransparency = 1}) end
                end
                local matchName = btn.Name
                for _, page in ipairs(allPages) do
                    local match = page:FindFirstChild(matchName)
                    if match and toggleStates[match] ~= nil then
                        toggleStates[match] = isActive
                        setToggleVisual(match, isActive)
                    end
                end
            end)
        end
    end
end

--------------------------------------------------------------------------------
-- WIRE ALL PAGES
--------------------------------------------------------------------------------
local allPages = {Movement, Combat, Keybinds, Controller, Utility, Settings}
for _, page in ipairs(allPages) do
    wireToggles(page)
    wireArrowToggles(page)
    wireExpandables(page)
    wireModeRows(page)
    wireKeybindRows(page)
    wireValueBoxes(page)
    wirePickerRows(page)
    wireThumbnailGalleries(page)
    wireActionButtons(page)
end
wireColorThemePicker(Settings)
wireMobileButtons()

--------------------------------------------------------------------------------
-- TAB SWITCHING
--------------------------------------------------------------------------------
local tabPages = {
    Movement = Movement,
    Combat = Combat,
    Keybinds = Keybinds,
    Controller = Controller,
    Utility = Utility,
    Settings = Settings,
}

local function switchTab(tabName)
    for _, page in pairs(tabPages) do page.Visible = false end
    if tabPages[tabName] then tabPages[tabName].Visible = true end
    for _, btn in ipairs(Tabs:GetChildren()) do
        if btn:IsA("TextButton") then
            local isActive = (btn.Name == tabName)
            btn.BackgroundTransparency = isActive and 0.78 or 1
            btn.TextColor3 = isActive and WHITE or MUTED_TEXT
            local stroke = btn:FindFirstChildWhichIsA("UIStroke")
            if stroke then
                local grad = stroke:FindFirstChildWhichIsA("UIGradient")
                if grad then
                    if isActive then
                        grad.Color = BORDER_GRAD_LIGHT.Color
                        grad.Transparency = BORDER_GRAD_LIGHT.Transparency
                    else
                        grad.Color = BORDER_GRAD_DARK.Color
                        grad.Transparency = BORDER_GRAD_DARK.Transparency
                    end
                end
            end
        end
    end
end

for _, btn in ipairs(Tabs:GetChildren()) do
    if btn:IsA("TextButton") then
        btn.MouseButton1Click:Connect(function()
            switchTab(btn.Name)
        end)
    end
end

--------------------------------------------------------------------------------
-- CLOSE / OPEN
--------------------------------------------------------------------------------
Close.MouseButton1Click:Connect(function()
    Main.Visible = false
    HenikaFloatOpen.Visible = true
end)

local floatBtn = HenikaFloatOpen:FindFirstChild("ImageButton")
if floatBtn then
    floatBtn.MouseButton1Click:Connect(function()
        Main.Visible = true
        HenikaFloatOpen.Visible = false
    end)
end

--------------------------------------------------------------------------------
-- INTRO SKIP
--------------------------------------------------------------------------------
local introSkipped = false
local introFinished = false

local function skipIntro()
    if introSkipped then return end
    introSkipped = true
    tween(HenikaIntro, 0.4, {BackgroundTransparency = 1})
    for _, child in ipairs(HenikaIntro:GetChildren()) do
        if child:IsA("ImageLabel") then tween(child, 0.3, {ImageTransparency = 1})
        elseif child:IsA("TextLabel") then tween(child, 0.3, {TextTransparency = 1})
        elseif child:IsA("Frame") then
            tween(child, 0.3, {BackgroundTransparency = 1})
            for _, sub in ipairs(child:GetDescendants()) do
                if sub:IsA("ImageLabel") then tween(sub, 0.3, {ImageTransparency = 1}) end
            end
        end
    end
    task.delay(0.45, function()
        introFinished = true
        HenikaIntro.Visible = false
        Main.BackgroundTransparency = 0
        local mainScale = Main:FindFirstChild("UIScale")
        if mainScale then
            mainScale.Scale = 0.85
            tween(mainScale, 0.35, {Scale = 1}, Enum.EasingStyle.Back)
        end
        HenikaFloatOpen.Visible = false
    end)
end

if TapCatcher then
    TapCatcher.MouseButton1Click:Connect(skipIntro)
end

task.spawn(function()
    task.wait(0.2)
    if introSkipped then return end
    task.wait(1.5)
    if not introSkipped then skipIntro() end
end)

--------------------------------------------------------------------------------
-- BAT AIMBOT & TP BAT FUNCTIONALITY (EXAMPLE IMPLEMENTATION)
--------------------------------------------------------------------------------
-- These functions can be customized for your specific game.
-- They are triggered by the toggles and use the selected mode.

local function getNearestEnemy()
    local nearest = nil
    local minDist = math.huge
    local char = LocalPlayer.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                local dist = (root.Position - targetRoot.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    nearest = player
                end
            end
        end
    end
    return nearest
end

local batAimbotActive = false
local tpBatActive = false

-- Called when Bat Aimbot toggle changes
local function onBatAimbotToggle(state)
    batAimbotActive = state
    if state then
        -- Start aiming loop
        task.spawn(function()
            while batAimbotActive and RunService.Run do
                local target = getNearestEnemy()
                if target and target.Character then
                    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
                    local char = LocalPlayer.Character
                    if char and targetRoot then
                        local root = char:FindFirstChild("HumanoidRootPart")
                        if root then
                            -- Aim at target
                            local lookAt = CFrame.lookAt(root.Position, targetRoot.Position)
                            root.CFrame = lookAt
                            -- If mode V2, maybe add extra behavior
                            if batAimbotMode == "V2" then
                                -- V2 specific: faster auto-swing or additional lock
                                -- Placeholder: maybe increase sensitivity
                            end
                        end
                    end
                end
                RunService.Heartbeat:Wait()
            end
        end)
    end
end

-- Called when TP Bat toggle changes
local function onTPBatToggle(state)
    tpBatActive = state
    if state then
        task.spawn(function()
            while tpBatActive and RunService.Run do
                local target = getNearestEnemy()
                if target and target.Character then
                    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
                    local char = LocalPlayer.Character
                    if char and targetRoot then
                        local root = char:FindFirstChild("HumanoidRootPart")
                        if root then
                            -- Teleport to target's position (maybe offset to avoid anti-bat)
                            -- Anti-bat counter: teleport slightly above or to the side
                            local tpPos = targetRoot.Position + Vector3.new(0, 2, 0)  -- adjust as needed
                            root.CFrame = CFrame.new(tpPos)
                        end
                    end
                end
                RunService.Heartbeat:Wait()
            end
        end)
    end
end

-- Hook toggles to functions
-- We need to find the Bat Aimbot and TP Bat rows in Combat tab
local batAimbotRow = Combat:FindFirstChild("Bat Aimbot")
local tpBatRow = Combat:FindFirstChild("TP Bat")

if batAimbotRow then
    -- Override toggle to include our function
    local function toggleBatAimbot(row)
        local state = toggleRow(row)
        onBatAimbotToggle(state)
        return state
    end
    -- Replace the toggle behavior
    for _, child in ipairs(batAimbotRow:GetChildren()) do
        if child:IsA("TextButton") and child.Name == "TextButton" then
            local clickCatcher = child:FindFirstChild("TextButton")
            if clickCatcher and clickCatcher:IsA("TextButton") then
                clickCatcher.MouseButton1Click:Connect(function()
                    toggleBatAimbot(batAimbotRow)
                end)
            else
                child.MouseButton1Click:Connect(function()
                    toggleBatAimbot(batAimbotRow)
                end)
            end
            break
        end
    end
end

if tpBatRow then
    local function toggleTPBat(row)
        local state = toggleRow(row)
        onTPBatToggle(state)
        return state
    end
    for _, child in ipairs(tpBatRow:GetChildren()) do
        if child:IsA("TextButton") and child.Name == "TextButton" then
            local clickCatcher = child:FindFirstChild("TextButton")
            if clickCatcher and clickCatcher:IsA("TextButton") then
                clickCatcher.MouseButton1Click:Connect(function()
                    toggleTPBat(tpBatRow)
                end)
            else
                child.MouseButton1Click:Connect(function()
                    toggleTPBat(tpBatRow)
                end)
            end
            break
        end
    end
end

--------------------------------------------------------------------------------
-- RETURN
--------------------------------------------------------------------------------
return {
    Gui = HenikaHub,
    Intro = HenikaIntro,
    Main = Main,
    Content = Content,
    Tabs = Tabs,
    FloatOpen = HenikaFloatOpen,
    MobileButtons = MobileButtons,
    Close = Close,
    toggleStates = toggleStates,
    keybindStates = keybindStates,
    switchTab = switchTab,
    skipIntro = skipIntro,
    toggleRow = toggleRow,
    batAimbotMode = batAimbotMode,
    setBatAimbotMode = function(mode) batAimbotMode = mode end,
}
