import pygame
import random
import os

# Initialize Pygame
pygame.init()

# Set up the game window
WINDOW_WIDTH = 800
WINDOW_HEIGHT = 600
window = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Call of Duty in 2D")

# Set the maximum frame rate
clock = pygame.time.Clock(120)
MAX_FPS = 120

# Define colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
BLOOD_RED = (128, 0, 0)

# Define fonts
font_small = pygame.font.Font(None, 24)
font_large = pygame.font.Font(None, 48)

# Load background image
original_background_image = pygame.image.load(os.path.join("airadventurelevel4.png"))
background_image = pygame.transform.scale(original_background_image, (WINDOW_WIDTH, WINDOW_HEIGHT))

# Load player sprite
DESIRED_PLAYER_WIDTH = 50
DESIRED_PLAYER_HEIGHT = 100
player_sprite = pygame.image.load(os.path.join("spriteGhost.png")).convert_alpha()
player_sprite = pygame.transform.scale(player_sprite, (DESIRED_PLAYER_WIDTH, DESIRED_PLAYER_HEIGHT))
PLAYER_WIDTH = player_sprite.get_width()
PLAYER_HEIGHT = player_sprite.get_height()

# Load enemy sprite
original_enemy_sprite = pygame.image.load(os.path.join("L9BqEs.png")).convert_alpha()
DESIRED_ENEMY_WIDTH = 75
DESIRED_ENEMY_HEIGHT = 75
enemy_sprite = pygame.transform.scale(original_enemy_sprite, (DESIRED_ENEMY_WIDTH, DESIRED_ENEMY_HEIGHT))
ENEMY_WIDTH = enemy_sprite.get_width()
ENEMY_HEIGHT = enemy_sprite.get_height()

# Load background music
pygame.mixer.music.load(os.path.join("Blindspots.mp3"))
pygame.mixer.music.set_volume(0.2)
pygame.mixer.music.play(-1)  # -1 means loop indefinitely

# Load the intro video
intro_video = pygame.movie.Movie(os.path.join("18df85f43b555-master_playlist"))
intro_video.set_display(window, (0, 0))

# Play the intro video
intro_video.play()

# Game properties
GAME_SPEED = 5
score = 0
data_corrupted = random.choice([True, False])
show_intro = True

# Game loop
running = True
while running:
    # Handle events
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if show_intro and event.key == pygame.K_ESCAPE:
                # Skip the intro video
                intro_video.stop()
                show_intro = False

    # Show the intro video
    if show_intro:
        # Check if the intro video has finished playing
        if not intro_video.get_busy():
            show_intro = False

        # Update the intro video
        intro_video.render_frame()
        continue

    # Player properties
    player_x = 50
    player_y = WINDOW_HEIGHT - PLAYER_HEIGHT - 100
    PLAYER_GRAVITY = 0.5
    player_velocity_x = 0
    player_velocity_y = 0
    is_jumping = False
    is_game_over = False
    PLAYER_SPEED = 5

    # Enemy properties
    enemies = []
    ENEMY_SPAWN_INTERVAL = 120  # Adjust this value to control enemy spawn rate
    enemy_spawn_cooldown = 0
    ENEMY_SPEED = 3

    # Get the keys that are currently pressed
    keys = pygame.key.get_pressed()

    # Game logic
    if not data_corrupted and not is_game_over:
        # Update player position
        player_velocity_x = 0
        if keys[pygame.K_LEFT]:
            player_velocity_x = -PLAYER_SPEED
        if keys[pygame.K_RIGHT]:
            player_velocity_x = PLAYER_SPEED

        player_x += player_velocity_x
        player_velocity_y += PLAYER_GRAVITY
        player_y += player_velocity_y

        # Check for collision with the ground
        if player_y + PLAYER_HEIGHT >= WINDOW_HEIGHT:
            player_y = WINDOW_HEIGHT - PLAYER_HEIGHT
            player_velocity_y = 0
            is_jumping = False

        # Keep player within the window bounds
        player_x = max(0, min(player_x, WINDOW_WIDTH - PLAYER_WIDTH))

        # Spawn new enemies
        enemy_spawn_cooldown += 1
        if enemy_spawn_cooldown >= ENEMY_SPAWN_INTERVAL:
            enemy_spawn_cooldown = 0
            enemy_x = WINDOW_WIDTH
            enemy_y = WINDOW_HEIGHT - ENEMY_HEIGHT - 100
            enemies.append((enemy_x, enemy_y))

        # Move enemies and check for collision
        for i, (enemy_x, enemy_y) in enumerate(enemies):
            enemy_x -= ENEMY_SPEED
            enemies[i] = (enemy_x, enemy_y)

            # Check for collision with player
            if (
                player_x + PLAYER_WIDTH > enemy_x
                and player_x < enemy_x + ENEMY_WIDTH
                and player_y + PLAYER_HEIGHT > enemy_y
            ):
                is_game_over = True

            # Remove enemies that have gone off-screen
            if enemy_x + ENEMY_WIDTH < 0:
                enemies.pop(i)

        # Clear the window
        window.blit(background_image, (0, 0))

        # Draw the player sprite
        window.blit(player_sprite, (player_x, player_y))

        # Draw enemies
        for enemy_x, enemy_y in enemies:
            window.blit(enemy_sprite, (enemy_x, enemy_y))

        # Draw score
        score_text = font_small.render(f"Score: {score}", True, WHITE)
        window.blit(score_text, (10, 10))

    elif is_game_over:
        # No "Game Over" message to be displayed
        pass

    else:
        # Draw the corrupted data message
        window.fill(BLACK)
        text = font_small.render("The data for Call of Duty In 2D is corrupted. Delete the game from your PC.", True, WHITE)
        text_rect = text.get_rect()
        text_rect.center = (WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
        window.blit(text, text_rect)

    # Update the display
    pygame.display.update()

    # Limit the frame rate
    clock.tick(MAX_FPS)

# Quit
pygame.quit()
