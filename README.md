import pygame
import sys
import random

# Inicialização do Pygame
pygame.init()

# Configurações da tela
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Vinicius: Life Adventure")

# Cores
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (50, 50, 255)
YELLOW = (255, 255, 0)

# FPS
clock = pygame.time.Clock()
FPS = 60

# Fontes
font = pygame.font.SysFont("Arial", 24)

# Carregar imagens
player_img = pygame.image.load("vinicius.png")  # personagem
family_img = pygame.image.load("family.png")    # família
friend_img = pygame.image.load("friends.png")   # amigos
enemy_img = pygame.image.load("enemy.png")      # inimigo
ball_img = pygame.image.load("ball.png")        # bola de futebol
medal_img = pygame.image.load("medal.png")      # medalha
memory_img = pygame.image.load("memory.png")    # lembrança

# Player
player_width, player_height = 50, 60
player_x, player_y = 100, HEIGHT - player_height - 50
player_vel = 5
jump = False
jump_height = 10
gravity = 0.5
y_velocity = 0
life = 3
score = 0

# Fases
current_level = 1
max_levels = 5

# Inimigos e itens
enemies = []
items = []

def spawn_enemies():
    for _ in range(current_level + 2):
        x = random.randint(200, WIDTH - 50)
        y = HEIGHT - player_height - 50
        enemies.append(pygame.Rect(x, y, 40, 40))

def spawn_items():
    for _ in range(3):
        x = random.randint(200, WIDTH - 50)
        y = HEIGHT - player_height - 50
        items.append(pygame.Rect(x, y, 30, 30))

spawn_enemies()
spawn_items()

# Cutscenes
cutscenes = {
    1: ["Childhood: Vinicius loves football!", "He has a family and a dog called Lasse."],
    2: ["School: Makes friends but distances from a mean boy."],
    3: ["Teenager: Trains hard and faces challenges."],
    4: ["Adult: Wins matches, suffers injuries, experiences love and loss."],
    5: ["Old Age: Reflects on life, family, friends. Peaceful end."]
}
cutscene_index = 0
show_cutscene = True

# Função para desenhar player
def draw_player(x, y):
    screen.blit(player_img, (x, y))

# Função para desenhar inimigos
def draw_enemies():
    for enemy in enemies:
        screen.blit(enemy_img, (enemy.x, enemy.y))

# Função para desenhar itens
def draw_items():
    for item in items:
        # Alterna entre bola, medalha e lembrança
        img = random.choice([ball_img, medal_img, memory_img])
        screen.blit(img, (item.x, item.y))

# Função para cutscene
def draw_cutscene(level, index):
    screen.fill(WHITE)
    if level in cutscenes:
        text = font.render(cutscenes[level][index], True, BLACK)
        screen.blit(text, (50, HEIGHT // 2))
    pygame.display.update()

# Loop principal do jogo
running = True
while running:
    clock.tick(FPS)

    if show_cutscene:
        draw_cutscene(current_level, cutscene_index)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                cutscene_index += 1
                if cutscene_index >= len(cutscenes[current_level]):
                    cutscene_index = 0
                    show_cutscene = False
        continue

    screen.fill(WHITE)

    # Eventos
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Controles
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:
        player_x -= player_vel
    if keys[pygame.K_RIGHT]:
        player_x += player_vel
    if not jump and keys[pygame.K_SPACE]:
        jump = True
        y_velocity = -jump_height

    # Gravidade
    if jump:
        player_y += y_velocity
        y_velocity += gravity
        if player_y >= HEIGHT - player_height - 50:
            player_y = HEIGHT - player_height - 50
            jump = False

    # Colisão com inimigos
    player_rect = pygame.Rect(player_x, player_y, player_width, player_height)
    for enemy in enemies:
        if player_rect.colliderect(enemy):
            life -= 1
            enemies.remove(enemy)

    # Colisão com itens
    for item in items:
        if player_rect.colliderect(item):
            score += 10
            items.remove(item)

    # Avançar fase
    if player_x + player_width >= WIDTH:
        player_x = 0
        current_level += 1
        if current_level > max_levels:
            print(f"Game Over! Final Score: {score}")
            running = False
        else:
            enemies.clear()
            items.clear()
            spawn_enemies()
            spawn_items()
            show_cutscene = True

    # Desenhar player, inimigos e itens
    draw_player(player_x, player_y)
    draw_enemies()
    draw_items()

    # Mostrar vida e pontuação
    life_text = font.render(f"Life: {life}", True, RED)
    score_text = font.render(f"Score: {score}", True, BLUE)
    screen.blit(life_text, (10, 10))
    screen.blit(score_text, (10, 40))

    # Fim do jogo se vida zerar
    if life <= 0:
        print(f"Vinicius couldn't complete his life story. Score: {score}")
        running = False

    pygame.display.update()

pygame.quit()
sys.exit()
