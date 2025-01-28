# 1st-Game
import pygame
import random
import os
import sys
from enum import Enum

# Initialize Pygame
pygame.init()

# Constants
WINDOW_WIDTH = 800
WINDOW_HEIGHT = 600
FPS = 60

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (34, 139, 34)
BLUE = (65, 105, 225)
BROWN = (139, 69, 19)
YELLOW = (255, 223, 0)

class GameState(Enum):
    MENU = 1
    PLAYING = 2
    GAME_OVER = 3

class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((30, 50))
        self.image.fill(BROWN)
        self.rect = self.image.get_rect()
        self.rect.center = (WINDOW_WIDTH // 2, WINDOW_HEIGHT - 100)
        self.speed = 5
        self.health = 100
        self.water = 100
        self.items = []
        self.score = 0

    def update(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and self.rect.left > 0:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT] and self.rect.right < WINDOW_WIDTH:
            self.rect.x += self.speed

class Item(pygame.sprite.Sprite):
    def __init__(self, item_type):
        super().__init__()
        self.type = item_type
        self.image = pygame.Surface((20, 20))
        if item_type == 'water':
            self.image.fill(BLUE)
        elif item_type == 'food':
            self.image.fill(GREEN)
        elif item_type == 'treasure':
            self.image.fill(YELLOW)
        self.rect = self.image.get_rect()
        self.rect.x = random.randint(0, WINDOW_WIDTH - 20)
        self.rect.y = -20
        self.speed = random.randint(2, 5)

    def update(self):
        self.rect.y += self.speed
        if self.rect.top > WINDOW_HEIGHT:
            self.kill()

class Game:
    def __init__(self):
        self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
        pygame.display.set_caption("Outdoor Adventure")
        self.clock = pygame.time.Clock()
        self.font = pygame.font.Font(None, 36)
        self.state = GameState.MENU
        self.background = self.create_background()
        
        # Sprite groups
        self.all_sprites = pygame.sprite.Group()
        self.items = pygame.sprite.Group()
        self.player = Player()
        self.all_sprites.add(self.player)
        
        # Game variables
        self.item_spawn_timer = 0
        self.resource_decay_timer = 0

    def create_background(self):
        # Create a simple nature background
        background = pygame.Surface((WINDOW_WIDTH, WINDOW_HEIGHT))
        
        # Sky
        background.fill((135, 206, 235))
        
        # Mountains
        pygame.draw.polygon(background, (169, 169, 169), 
                          [(0, 400), (200, 200), (400, 400)])
        pygame.draw.polygon(background, (128, 128, 128), 
                          [(300, 400), (500, 150), (700, 400)])
        
        # Ground
        pygame.draw.rect(background, GREEN, (0, 400, WINDOW_WIDTH, 200))
        
        return background

    def spawn_item(self):
        item_type = random.choice(['water', 'food', 'treasure'])
        new_item = Item(item_type)
        self.all_sprites.add(new_item)
        self.items.add(new_item)

    def draw_stats(self):
        # Draw health bar
        pygame.draw.rect(self.screen, (255, 0, 0), (10, 10, 200, 20))
        pygame.draw.rect(self.screen, (0, 255, 0), 
                        (10, 10, 200 * (self.player.health/100), 20))
        
        # Draw water bar
        pygame.draw.rect(self.screen, (100, 100, 255), (10, 40, 200, 20))
        pygame.draw.rect(self.screen, (0, 0, 255), 
                        (10, 40, 200 * (self.player.water/100), 20))
        
        # Draw score
        score_text = self.font.render(f'Score: {self.player.score}', True, BLACK)
        self.screen.blit(score_text, (10, 70))

    def draw_menu(self):
        self.screen.fill((135, 206, 235))
        title = self.font.render('Outdoor Adventure', True, BLACK)
        start_text = self.font.render('Press SPACE to Start', True, BLACK)
        self.screen.blit(title, (WINDOW_WIDTH//2 - title.get_width()//2, 200))
        self.screen.blit(start_text, 
                        (WINDOW_WIDTH//2 - start_text.get_width()//2, 300))

    def draw_game_over(self):
        self.screen.fill((135, 206, 235))
        game_over_text = self.font.render('Game Over!', True, BLACK)
        score_text = self.font.render(f'Final Score: {self.player.score}', 
                                    True, BLACK)
        restart_text = self.font.render('Press R to Restart', True, BLACK)
        
        self.screen.blit(game_over_text, 
                        (WINDOW_WIDTH//2 - game_over_text.get_width()//2, 200))
        self.screen.blit(score_text, 
                        (WINDOW_WIDTH//2 - score_text.get_width()//2, 300))
        self.screen.blit(restart_text, 
                        (WINDOW_WIDTH//2 - restart_text.get_width()//2, 400))

    def reset_game(self):
        self.all_sprites.empty()
        self.items.empty()
        self.player = Player()
        self.all_sprites.add(self.player)
        self.state = GameState.PLAYING

    def run(self):
        running = True
        while running:
            # Event handling
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_SPACE and self.state == GameState.MENU:
                        self.state = GameState.PLAYING
                    elif event.key == pygame.K_r and self.state == GameState.GAME_OVER:
                        self.reset_game()

            if self.state == GameState.PLAYING:
                # Spawn items
                self.item_spawn_timer += 1
                if self.item_spawn_timer >= 60:
                    self.spawn_item()
                    self.item_spawn_timer = 0

                # Update
                self.all_sprites.update()

                # Resource decay
                self.resource_decay_timer += 1
                if self.resource_decay_timer >= 60:
                    self.player.health -= 1
                    self.player.water -= 1.5
                    self.resource_decay_timer = 0

                # Collision detection
                hits = pygame.sprite.spritecollide(self.player, self.items, True)
                for hit in hits:
                    if hit.type == 'water':
                        self.player.water = min(100, self.player.water + 20)
                    elif hit.type == 'food':
                        self.player.health = min(100, self.player.health + 15)
                    elif hit.type == 'treasure':
                        self.player.score += 100

                # Check game over condition
                if self.player.health <= 0 or self.player.water <= 0:
                    self.state = GameState.GAME_OVER

                # Draw
                self.screen.blit(self.background, (0, 0))
                self.all_sprites.draw(self.screen)
                self.draw_stats()

            elif self.state == GameState.MENU:
                self.draw_menu()
            elif self.state == GameState.GAME_OVER:
                self.draw_game_over()

            # Update display
            pygame.display.flip()
            self.clock.tick(FPS)

        pygame.quit()
        sys.exit()

if __name__ == "__main__":
    game = Game()
    game.run()
