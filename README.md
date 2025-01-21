# Space-War
# Importing necessary libraries
import pygame   #Creating window, showing title
import random   #For random movements of modules
from collections import deque   #For adding and removing elements from both beginning and end of queue

# Initializing pygame
pygame.init()

# Defining constants for screen width & height
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600

# Setting the caption of the game window
pygame.display.set_caption("Space Shooter")

 # Defining Constants for colours
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

# Loading game assets(images)
spaceship_image = pygame.image.load("C:/Users/HP/Downloads/spaceship.jpg")
spaceship_image = pygame.transform.scale(spaceship_image, (60, 60))

ally_image = pygame.image.load("C:/Users/HP/Downloads/ally.png")
ally_image = pygame.transform.scale(ally_image, (60, 60))

bullet_image = pygame.image.load("C:/Users/HP/Downloads/bullet.jpg")
bullet_image = pygame.transform.scale(bullet_image, (5, 10))

asteroid_image = pygame.image.load("C:/Users/HP/Downloads/asteroid.png")
asteroid_image = pygame.transform.scale(asteroid_image,  (50, 50))

explosion_image = pygame.image.load("C:/Users/HP/Downloads/explosion (1).jpg")
explosion_image = pygame.transform.scale(explosion_image, (100, 100))

background_image = pygame.image.load("C:/Users/HP/Downloads/background.jpg")
background_image = pygame.transform.scale(background_image, (SCREEN_WIDTH, SCREEN_HEIGHT))

# Initializing Sound Mixer
pygame.mixer.init()

# Loading Sound Effects
fire_sound = pygame.mixer.Sound("C:/Users/HP/Downloads/DS GAME/laser-gun.wav")
explosion_sound = pygame.mixer.Sound("C:/Users/HP/Downloads/DS GAME/explosion.wav")

# Initializing Game variables
spaceship_x = SCREEN_WIDTH // 2 #Position
spaceship_y = SCREEN_HEIGHT - 80
spaceship_speed = 5

ally_x = SCREEN_WIDTH // 4
ally_y = SCREEN_HEIGHT - 80
ally_speed = 5

lives = 5
score = 0

# Creating a font object
font = pygame.font.Font(None, 36)

#Initialize bullet, asteroid,explosion,allies, ally_bullets lists
bullets = deque()
asteroids = deque()
explosions = []
allies = [[ally_x, ally_y]]
ally_bullets = []


# Initializing Game difficulty variables
hit_count = 0
asteroid_speed = 5
spawn_interval = 50 #Asteroid appearing interval

# Creating Clock Object
clock = pygame.time.Clock()

# Defining Functions to draw game objects
def draw_bullets():
    for bullet in bullets:
        screen.blit(bullet_image, (bullet[0], bullet[1]))

def draw_asteroids():
    for asteroid in asteroids:
        screen.blit(asteroid_image, (asteroid[0], asteroid[1]))

def draw_explosions():
    for explosion in list(explosions):
        screen.blit(explosion_image, (explosion[0], explosion[1]))
        explosion[2] -= 1
        if explosion[2] <= 0:
            explosions.remove(explosion)

def draw_allies():
    for ally in allies:
        screen.blit(ally_image, (ally[0], ally[1]))

def display_lives():
    lives_text = font.render(f"Lives: {lives}", True, WHITE)
    screen.blit(lives_text, (SCREEN_WIDTH - 120, 10))

# Initializing screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Space Shooter")

# Game loop
running = True
while running:
    # Handling Events
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
    #Get key presses
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT] and spaceship_x > 0:
        spaceship_x -= spaceship_speed
    if keys[pygame.K_RIGHT] and spaceship_x < SCREEN_WIDTH - 60:
        spaceship_x += spaceship_speed
    if keys[pygame.K_SPACE]:
        bullets.append([spaceship_x + 25, spaceship_y])
        fire_sound.play()

    # Ally shoots bullets
    for ally in allies:
        for asteroid in asteroids:
            if abs(ally[0] - asteroid[0]) < 200 and abs(ally[1] - asteroid[1]) < 200:
                if random.randint(1, 100) < 10:  # 10% chance of firing
                    ally_bullets.append([ally[0] + 25, ally[1]])
                    fire_sound.play()

    # Move allies
    for ally in allies:
        if keys[pygame.K_a] and ally[0] > 0:
            ally[0] -= ally_speed
        if keys[pygame.K_d] and ally[0] < SCREEN_WIDTH - 60:
            ally[0] += ally_speed
        if keys[pygame.K_w] and ally[1] > 0:
            ally[1] -= ally_speed
        if keys[pygame.K_s] and ally[1] < SCREEN_HEIGHT - 60:
            ally[1] += ally_speed

    # Update bullets
    for bullet in bullets:
        bullet[1] -= 10
    bullets = deque([bullet for bullet in bullets if bullet[1] > 0])

    #Update Ally bullets
    for bullet in ally_bullets:
        bullet[1] -= 10
    ally_bullets = [bullet for bullet in ally_bullets if bullet[1] > 0]

    # Spawn asteroids
    if random.randint(1, spawn_interval) == 1:
        asteroids.append([random.randint(0, SCREEN_WIDTH - 50), -50])

    # Update asteroids
    for asteroid in asteroids:
        asteroid[1] += asteroid_speed
    asteroids = deque([asteroid for asteroid in asteroids if asteroid[1] < SCREEN_HEIGHT])

    # Check collisions
    for bullet in list(bullets):
        for asteroid in list(asteroids):
            if (bullet[0] < asteroid[0] + 50 and
                bullet[0] + 10 > asteroid[0] and
                bullet[1] < asteroid[1] + 50 and
                bullet[1] + 30 > asteroid[1]):
                bullets.remove(bullet)
                asteroids.remove(asteroid)
                score += 10
                hit_count += 1
                explosion_sound.play()
                explosions.append([asteroid[0], asteroid[1], 30])

    # Check collisions for ally bullets
    for bullet in list(ally_bullets):
        for asteroid in list(asteroids):
            if (bullet[0] < asteroid[0] + 50 and
                    bullet[0] + 10 > asteroid[0] and
                    bullet[1] < asteroid[1] + 50 and
                    bullet[1] + 30 > asteroid[1]):
                ally_bullets.remove(bullet)
                asteroids.remove(asteroid)
                explosion_sound.play()
                explosions.append([asteroid[0], asteroid[1], 30])

    # Increase difficulty
    if hit_count >= 10:
        asteroid_speed = 7
        spawn_interval = 40

    # Check for spaceship collisions
    for asteroid in list(asteroids):
        if (spaceship_x < asteroid[0] + 50 and
                spaceship_x + 60 > asteroid[0] and
                spaceship_y < asteroid[1] + 50 and
                spaceship_y + 60 > asteroid[1]):
            asteroids.remove(asteroid)
            lives -= 1
            explosion_sound.play()
            explosions.append([spaceship_x, spaceship_y, 30])

    # Check for ally collisions
    for ally in allies:
        for asteroid in list(asteroids):
            if (ally[0] < asteroid[0] + 50 and
                    ally[0] + 60 > asteroid[0] and
                    ally[1] < asteroid[1] + 50 and
                    ally[1] + 60 > asteroid[1]):
                asteroids.remove(asteroid)
                lives -= 1
                explosion_sound.play()
                explosions.append([ally[0], ally[1], 30])

    # Game over
    if lives == 0:
        running = False

    # Draw everything
    screen.blit(background_image, (0, 0))
    screen.blit(spaceship_image, (spaceship_x, spaceship_y))
    draw_bullets()
    for bullet in ally_bullets:
        screen.blit(bullet_image, (bullet[0], bullet[1]))

    draw_asteroids()
    draw_explosions()
    draw_allies()

    # Display score and lives
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))
    display_lives()

    # Update display
    pygame.display.flip()

    # Cap the frame rate
    clock.tick(60)


