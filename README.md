import pygame
import sys

# ИНИЦИАЛИЗАЦИЯ
pygame.init()
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Маг против Ведьмы и Босса")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 32)
big_font = pygame.font.SysFont(None, 72)

# ЦВЕТА
WHITE = (255, 255, 255)
RED = (255, 0, 0)
BLUE = (50, 50, 255)
GREEN = (0, 255, 0)
BLACK = (0, 0, 0)
PURPLE = (150, 0, 150)
YELLOW = (255, 255, 0)

# ПЕРЕМЕННЫЕ ИГРОКА
player = pygame.Rect(100, 300, 40, 40)
player_speed = 6
player_bullets = []
player_health = 5
player_max_health = 5
shoot_cooldown = 0

# ВЕДЬМА
witch = pygame.Rect(400, 100, 40, 40)
witch_bullets = []
witch_cooldown = 0
witch_health = 8
witch_max_health = 8
witch_direction = 1
witch_speed = 3

# БОСС
boss = pygame.Rect(650, 400, 60, 60)
boss_bullets = []
boss_cooldown = 0
boss_direction = -1
boss_speed = 2
boss_health = 15
boss_max_health = 15

# СТАТУС ИГРЫ
game_over = False
game_won = False

# ФУНКЦИЯ ОТРИСОВКИ ПОЛОС ЗДОРОВЬЯ
def draw_health_bar(x, y, health, max_health, width=60, height=10):
    ratio = health / max_health
    pygame.draw.rect(screen, RED, (x, y, width, height))
    pygame.draw.rect(screen, GREEN, (x, y, width * ratio, height))

# GAME LOOP
running = True
while running:
    clock.tick(60)
    screen.fill(BLACK)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    if not game_over and not game_won:
        # УПРАВЛЕНИЕ
        keys = pygame.key.get_pressed()
        if keys[pygame.K_w]: player.y -= player_speed
        if keys[pygame.K_s]: player.y += player_speed
        if keys[pygame.K_a]: player.x -= player_speed
        if keys[pygame.K_d]: player.x += player_speed

        # ВЫСТРЕЛ
        if shoot_cooldown > 0:
            shoot_cooldown -= 1
        if keys[pygame.K_SPACE] and shoot_cooldown == 0:
            if len(player_bullets) < 5:
                player_bullets.append(pygame.Rect(player.centerx, player.centery, 10, 5))
                shoot_cooldown = 15

        # ДВИЖЕНИЕ ВРАГОВ
        witch.y += witch_direction * witch_speed
        if witch.top <= 0 or witch.bottom >= HEIGHT:
            witch_direction *= -1

        boss.y += boss_direction * boss_speed
        if boss.top <= 0 or boss.bottom >= HEIGHT:
            boss_direction *= -1

        # ПУЛИ ИГРОКА
        for bullet in player_bullets[:]:
            bullet.x += 10
            if bullet.x > WIDTH:
                player_bullets.remove(bullet)
            elif bullet.colliderect(witch):
                witch_health -= 1
                player_bullets.remove(bullet)
            elif bullet.colliderect(boss):
                boss_health -= 2
                player_bullets.remove(bullet)

        # ПУЛИ ВЕДЬМЫ
        witch_cooldown += 1
        if witch_cooldown > 60:
            witch_bullets.append(pygame.Rect(witch.left, witch.centery, 10, 5))
            witch_cooldown = 0
        for bullet in witch_bullets[:]:
            bullet.x -= 6
            if bullet.colliderect(player):
                player_health -= 1
                witch_bullets.remove(bullet)
            elif bullet.x < 0:
                witch_bullets.remove(bullet)

        # ПУЛИ БОССА
        boss_cooldown += 1
        if boss_cooldown > 90:
            boss_bullets.append(pygame.Rect(boss.left, boss.centery, 20, 10))
            boss_cooldown = 0
        for bullet in boss_bullets[:]:
            bullet.x -= 3
            if bullet.colliderect(player):
                player_health -= 2
                boss_bullets.remove(bullet)
            elif bullet.x < 0:
                boss_bullets.remove(bullet)

        # ПРОВЕРКА ПРОИГРЫША/ПОБЕДЫ
        if player_health <= 0:
            game_over = True
        elif witch_health <= 0 and boss_health <= 0:
            game_won = True

    # ОТРИСОВКА
    pygame.draw.rect(screen, BLUE, player)
    pygame.draw.rect(screen, GREEN, witch)
    pygame.draw.rect(screen, PURPLE, boss)

    # Пули
    for bullet in player_bullets:
        pygame.draw.rect(screen, WHITE, bullet)
    for bullet in witch_bullets:
        pygame.draw.rect(screen, RED, bullet)
    for bullet in boss_bullets:
        pygame.draw.rect(screen, YELLOW, bullet)

    # Имена
    screen.blit(font.render("Маг", True, WHITE), (player.x, player.y - 20))
    screen.blit(font.render("Ведьма", True, WHITE), (witch.x, witch.y - 20))
    screen.blit(font.render("БОСС", True, WHITE), (boss.x, boss.y - 20))

    # Полоски здоровья
    draw_health_bar(player.x, player.y - 10, player_health, player_max_health, 40)
    draw_health_bar(witch.x, witch.y - 10, witch_health, witch_max_health, 40)
    draw_health_bar(boss.x, boss.y - 10, boss_health, boss_max_health, 60)

    # Сообщения об окончании
    if game_over:
        text = big_font.render("Ты проиграл!", True, RED)
        screen.blit(text, (WIDTH // 2 - text.get_width() // 2, HEIGHT // 2 - 40))
    elif game_won:
        text = big_font.render("Ты победил!", True, GREEN)
        screen.blit(text, (WIDTH // 2 - text.get_width() // 2, HEIGHT // 2 - 40))

    pygame.display.flip()

pygame.quit()
sys.exit()
