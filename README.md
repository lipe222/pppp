-- Script de teste do Aimbot para sistema anti-trapaça (tiros no corpo, rastreamento ultrassuave)
-- Características: Trava precisa, Rastreamento de fluidos, Filtro FOV, Ativação por chave, Suporte R6 e R15
-- AVISO: Use somente para fins de teste

Jogadores locais = jogo:GetService("Jogadores")
Serviço de Entrada de Usuário local = jogo:GetService("Serviço de Entrada de Usuário")
local RunService = jogo:GetService("RunService")

local LocalPlayer = Jogadores.LocalPlayer
Mouse local = LocalPlayer:GetMouse()
Câmera local = workspace.CurrentCamera

-- Configurações
local AimEnabled = falso
Suavidade local = 0,08 -- menor é mais lento/suave
local AimbotMode = "Instantâneo" -- "Instantâneo" ou "Suave"
local MaxFOV = 200 -- Raio máximo de pixel do mouse para considerar um alvo

-- Obtém a parte do corpo do personagem que pode ser mirada (para R6/R15)
função local getBody(caractere)
    retornar caractere:FindFirstChild("HumanoidRootPart") ou caractere:FindFirstChild("UpperTorso") ou caractere:FindFirstChild("Torso")
fim

-- Obtenha o jogador visível mais próximo dentro do campo de visão
função local getClosestPlayer()
    local mais próximo = nulo
    local mais curto = MaxFOV

    para _, jogador em ipairs(Players:GetPlayers()) faça
        se jogador ~= LocalPlayer e jogador.Character então
            personagem local = jogador.Personagem
            humanoide local = personagem:FindFirstChildOfClass("Humanoide")

            se humanoide e humanoide.Saúde > 0 então
                parte local = getBody(caractere)
                se parte então
                    screenPos local, onScreen = Câmera:WorldToScreenPoint(part.Position)
                    se na tela então
                        distância local = (Vector2.new(Mouse.X, Mouse.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                        se dist < mais curto então
                            mais curto = dist
                            mais próximo = parte
                        fim
                    fim
                fim
            fim
        fim
    fim

    retornar mais próximo
fim

-- Mira suave ou instantânea em direção ao alvo
função local aimAtTarget()
    targetPart local = getClosestPlayer()
    se targetPart então
        targetPos local = targetPart.Position
        camPos local = Câmera.CFrame.Posição
        aimCFrame local = CFrame.new(camPos, targetPos)

        se AimbotMode == "Instantâneo" então
            Câmera.CFrame = aimCFrame
        outro
            Camera.CFrame = Camera.CFrame:Lerp(aimCFrame, Suavidade)
        fim
    fim
fim

-- Alternar aimbot com clique direito
UserInputService.InputBegan:Connect(função(entrada, processado)
    se não for processado e input.UserInputType == Enum.UserInputType.MouseButton2 então
        AimEnabled = verdadeiro
    fim
fim)

UserInputService.InputEnded:Connect(função(entrada, processado)
    se input.UserInputType == Enum.UserInputType.MouseButton2 então
        AimEnabled = falso
    fim
fim)

-- Loop principal
RunService.RenderStepped:Connect(função()
    se AimEnabled então
        aimAtTarget()
    fim
fim)


-- Script ESP Universal (compatível com R6 + R15)
-- API de desenho | Limpeza na morte/reinicialização | Cor azul para todos os jogadores

Jogadores locais = jogo:GetService("Jogadores")
local RunService = jogo:GetService("RunService")
Câmera local = workspace.CurrentCamera
local LocalPlayer = Jogadores.LocalPlayer

ESPObjects locais = {}

-- Limpar ESP
função local removeESP(jogador)
	esp local = ESPObjects[jogador]
	se esp então
		se esp.Box então esp.Box:Remove() fim
		se esp.Name então esp.Name:Remove() fim
		ESPObjects[jogador] = nulo
	fim
fim

-- Criar visuais ESP
função local createESP(jogador)
	se jogador == LocalPlayer então retorne fim

	removeESP(jogador)

	caixa local = Desenho.new("Quadrado")
	caixa.Espessura = 1
	caixa.Preenchida = falso
	box.Color = Color3.fromRGB(0, 0, 255) -- Cor azul
	caixa.Transparência = 1
	caixa.Visível = falso

	nome localTag = Desenho.new("Texto")
	nameTag.Size = 14
	nameTag.Center = verdadeiro
	nameTag.Outline = verdadeiro
	nameTag.Color = Color3.fromRGB(0, 0, 255) -- Cor azul
	nameTag.Text = jogador.Nome
	nameTag.Visible = falso

	ESPObjects[jogador] = {
		Caixa = caixa,
		Nome = nameTag,
	}

	-- Lidar com o reaparecimento do personagem
	player.CharacterAdded:Connect(função()
		tarefa.espera(1)
		createESP(jogador)
	fim)

	player.CharacterRemoving:Connect(função()
		removeESP(jogador)
	fim)

	-- Lidar com a limpeza da morte
	se jogador.Personagem então
		humanoide local = player.Character:FindFirstChildOfClass("Humanoide")
		se humanoide então
			humanoid.Morreu:Conectar(função()
				removeESP(jogador)
			fim)
		fim
	fim
fim

-- Obter topo/fundo do personagem (R6 ou R15)
função local getCharacterBounds(caractere)
	partes locais = {}

	para _, parte em ipairs(character:GetChildren()) faça
		se part:IsA("BasePart") e part.Name ~= "HumanoidRootPart" então
			tabela.inserir(partes, parte)
		fim
	fim

	local superior, inferior = nulo, nulo

	para _, parte em ipairs(partes) faça
		se não for top ou part.Position.Y > top.Y então
			topo = parte.Posição
		fim
		se não for inferior ou parte.Posição.Y < inferior.Y então
			inferior = parte.Posição
		fim
	fim

	retornar para cima, para baixo
fim

-- Desenho de loop
RunService.RenderStepped:Connect(função()
	para o jogador, especialmente em pares (ESPObjects) faça
		personagem local = jogador.Personagem
		se personagem e personagem:FindFirstChild("HumanoidRootPart") então
			raiz local = personagem.HumanoidRootPart
			local superior, inferior = getCharacterBounds(caractere)
			se não for superior ou inferior, então continue até o final

			Tela superior local, tela superior = Câmera:WorldToViewportPoint(topo)
			Tela inferior local, tela inferior = Câmera:WorldToViewportPoint(inferior)

			se topOnScreen e bottomOnScreen então
				altura local = math.abs(telasuperior.Y - telainferior.Y)
				largura local = altura / 2
				centro localX = (telasuperior.X + telainferior.X) / 2

				esp.Box.Size = Vector2.new(largura, altura)
				esp.Box.Position = Vector2.new(centerX - largura / 2, topScreen.Y)
				esp.Box.Visible = verdadeiro

				esp.Nome.Texto = jogador.Nome
				esp.Nome.Posição = Vetor2.novo(centroX, telasuperior.Y - 16)
				esp.Nome.Visível = verdadeiro
			outro
				esp.Box.Visible = falso
				esp.Nome.Visível = falso
			fim
		outro
			se esp então
				esp.Box.Visible = falso
				esp.Nome.Visível = falso
			fim
		fim
	fim
fim)

-- Configuração inicial
para _, jogador em ipairs(Players:GetPlayers()) faça
	se jogador ~= LocalPlayer então
		createESP(jogador)
	fim
fim

Jogadores.JogadorAdicionado:Conectar(função(jogador)
	player.CharacterAdded:Connect(função()
		tarefa.espera(1)
		createESP(jogador)
	fim)
fim)

Jogadores.PlayerRemoving:Connect(removeESP)
