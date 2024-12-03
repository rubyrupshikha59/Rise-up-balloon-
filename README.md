# Rise-up-balloon-
import pygame
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Rise Up Balloon Game")

# Colors
WHITE = (255, 255, 255)
SKY_BLUE = (135, 206, 235)
BLACK = (0, 0, 0)

# Create a surface for the translucent balloon
balloon_radius = 40
balloon_surface = pygame.Surface((balloon_radius * 2, balloon_radius * 2), pygame.SRCALPHA)
pygame.draw.circle(balloon_surface, (255, 255, 255, 128), (balloon_radius, balloon_radius), balloon_radius)

balloon_x = WIDTH // 2
balloon_y = HEIGHT - 100

# String
string_length = 100

# Paddle (Circle Shield)
shield_radius = 20
shield_x = WIDTH // 2
shield_y = HEIGHT - 150

# Obstacles
def create_obstacle():
    shape_type = random.choice(["rect", "circle", "triangle"])
    width = random.randint(20, 70)
    height = random.randint(20, 70)
    x = random.randint(0, WIDTH - width)
    y = random.randint(-1500, -100)
    return {"shape": shape_type, "rect": pygame.Rect(x, y, width, height)}

obstacles = [create_obstacle() for _ in range(10)]
obstacle_speed = 2

# Game variables
running = True
clock = pygame.time.Clock()
shield_speed = 5
score = 0
font = pygame.font.SysFont(None, 55)

# Game Over variable
game_over = False

def move_obstacles():
    global score
    for obstacle in obstacles:
        obstacle["rect"].y += obstacle_speed
        if obstacle["rect"].y > HEIGHT:
            obstacles.remove(obstacle)
            obstacles.append(create_obstacle())
            score += 1

def draw():
    screen.fill(SKY_BLUE)  # Draw sky blue background
    screen.blit(balloon_surface, (balloon_x - balloon_radius, balloon_y - balloon_radius))
    pygame.draw.line(screen, BLACK, (balloon_x, balloon_y + balloon_radius), (balloon_x, balloon_y + balloon_radius + string_length), 2)  # Draw string
    pygame.draw.circle(screen, WHITE, (shield_x, shield_y), shield_radius)
    for obstacle in obstacles:
        if obstacle["shape"] == "rect":
            pygame.draw.rect(screen, WHITE, obstacle["rect"])
        elif obstacle["shape"] == "circle":
            pygame.draw.ellipse(screen, WHITE, obstacle["rect"])
        elif obstacle["shape"] == "triangle":
            points = [(obstacle["rect"].centerx, obstacle["rect"].y),
                      (obstacle["rect"].x, obstacle["rect"].bottom),
                      (obstacle["rect"].right, obstacle["rect"].bottom)]
            pygame.draw.polygon(screen, WHITE, points)
    draw_score()
    if game_over:
        show_game_over()
    pygame.display.flip()

def draw_score():
    score_text = font.render(f"Score: {score}", True, BLACK)
    screen.blit(score_text, (10, 10))

def show_game_over():
    game_over_text = font.render("Game Over!", True, BLACK)
    restart_text = font.render("Press R to Restart", True, BLACK)
    screen.blit(game_over_text, (WIDTH // 2 - 100, HEIGHT // 2 - 50))
    screen.blit(restart_text, (WIDTH // 2 - 150, HEIGHT // 2))

def restart_game():
    global score, game_over, obstacles
    score = 0
    game_over = False
    obstacles = [create_obstacle() for _ in range(10)]
    move_obstacles()

# Main game loop
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT] and shield_x - shield_radius > 0:
        shield_x -= shield_speed
    if keys[pygame.K_RIGHT] and shield_x + shield_radius < WIDTH:
        shield_x += shield_speed
    if keys[pygame.K_r] and game_over:
        restart_game()

    if not game_over:
        move_obstacles()

        for obstacle in obstacles:
            if pygame.Rect(balloon_x - balloon_radius, balloon_y - balloon_radius, balloon_radius * 2, balloon_radius * 2).colliderect(obstacle["rect"]):
                game_over = True
            if pygame.Rect(shield_x - shield_radius, shield_y - shield_radius, shield_radius * 2, shield_radius * 2).colliderect(obstacle["rect"]):
                obstacles.remove(obstacle)
                obstacles.append(create_obstacle())

    draw()
    clock.tick(120)

pygame.quit()
