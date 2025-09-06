‎import pygame
‎import random
‎import sys
‎import os
‎
‎# Game constants
‎WIDTH, HEIGHT = 800, 600
‎ROAD_W = 200
‎ROAD_H = HEIGHT
‎LANE_W = ROAD_W // 3
‎CAR_W, CAR_H = 50, 100
‎FPS = 60
‎
‎# Colors
‎WHITE = (255, 255, 255)
‎GRAY = (100, 100, 100)
‎GREEN = (34, 177, 76)
‎RED = (200, 0, 0)
‎YELLOW = (240, 240, 0)
‎BLUE = (0, 0, 255)
‎BLACK = (0, 0, 0)
‎PURPLE = (150, 0, 150)
‎ORANGE = (255, 120, 0)
‎
‎CAR_COLORS = [RED, BLUE, YELLOW, PURPLE, ORANGE]
‎CAR_NAMES = ["Ferrari", "Toyota", "hamburger", "Purple", "Orange"]
‎
‎HIGHSCORE_FILE = "highscore.txt"
‎
‎pygame.init()
‎screen = pygame.display.set_mode((WIDTH, HEIGHT))
‎pygame.display.set_caption("Pseudo-3D Racing Game")
‎clock = pygame.time.Clock()
‎font = pygame.font.SysFont(None, 36)
‎
‎def draw_road(level):
‎    pygame.draw.rect(screen, GRAY, ((WIDTH//2 - ROAD_W//2, 0), (ROAD_W, HEIGHT)))
‎    # Draw lane lines
‎    for i in range(0, HEIGHT, 40):
‎        for lane in range(1,3):
‎            pygame.draw.rect(screen, WHITE, (
‎                WIDTH//2 - ROAD_W//2 + lane*LANE_W - 5, i + (level*10)%40, 10, 30))
‎
‎def draw_car(x, y, color):
‎    pygame.draw.rect(screen, color, (x, y, CAR_W, CAR_H))
‎    pygame.draw.rect(screen, WHITE, (x+10, y+20, CAR_W-20, CAR_H//4))
‎    pygame.draw.circle(screen, BLACK, (x+10, y+CAR_H-10), 10)
‎    pygame.draw.circle(screen, BLACK, (x+CAR_W-10, y+CAR_H-10), 10)
‎
‎class Car:
‎    def __init__(self, lane, color, y):
‎        self.lane = lane
‎        self.color = color
‎        self.y = y
‎        self.speed = random.randint(5, 10)
‎    def update(self, level):
‎        self.y += self.speed + level
‎        if self.y > HEIGHT:
‎            self.y = -CAR_H
‎            self.lane = random.randint(0,2)
‎            self.speed = random.randint(5, 10)
‎    def draw(self):
‎        x = WIDTH//2 - ROAD_W//2 + self.lane*LANE_W + (LANE_W - CAR_W)//2
‎        draw_car(x, self.y, self.color)
‎
‎class PowerUp:
‎    def __init__(self):
‎        self.lane = random.randint(0,2)
‎        self.y = random.randint(-600, -100)
‎        self.active = True
‎    def update(self, level):
‎        self.y += 8 + level
‎        if self.y > HEIGHT:
‎            self.y = random.randint(-600, -100)
‎            self.lane = random.randint(0,2)
‎            self.active = True
‎    def draw(self):
‎        if self.active:
‎            x = WIDTH//2 - ROAD_W//2 + self.lane*LANE_W + (LANE_W - CAR_W)//2 + 10
‎            pygame.draw.circle(screen, ORANGE, (x+CAR_W//2, self.y+CAR_H//2), 20)
‎            pygame.draw.circle(screen, YELLOW, (x+CAR_W//2, self.y+CAR_H//2), 10)
‎
‎def get_highscore():
‎    if os.path.exists(HIGHSCORE_FILE):
‎        with open(HIGHSCORE_FILE, "r") as f:
‎            return int(f.read())
‎    return 0
‎
‎def save_highscore(score):
‎    with open(HIGHSCORE_FILE, "w") as f:
‎        f.write(str(score))
‎
‎def choose_car():
‎    selected = 0
‎    choosing = True
‎    while choosing:
‎        screen.fill(GREEN)
‎        title = font.render("Choose your car color!", True, WHITE)
‎        screen.blit(title, (WIDTH//2 - 150, 50))
‎        for idx, color in enumerate(CAR_COLORS):
‎            x = 100 + idx*140
‎            draw_car(x, HEIGHT//2, color)
‎            name = font.render(CAR_NAMES[idx], True, WHITE)
‎            screen.blit(name, (x, HEIGHT//2 + CAR_H + 10))
‎            if idx == selected:
‎                pygame.draw.rect(screen, YELLOW, (x-5, HEIGHT//2-5, CAR_W+10, CAR_H+10), 4)
‎        instruction = font.render("Use LEFT/RIGHT keys. Press ENTER to select.", True, WHITE)
‎        screen.blit(instruction, (WIDTH//2 - 250, HEIGHT - 100))
‎        for event in pygame.event.get():
‎            if event.type == pygame.QUIT:
‎                pygame.quit()
‎                sys.exit()
‎            elif event.type == pygame.KEYDOWN:
‎                if event.key == pygame.K_LEFT and selected > 0:
‎                    selected -= 1
‎                elif event.key == pygame.K_RIGHT and selected < len(CAR_COLORS)-1:
‎                    selected += 1
‎                elif event.key == pygame.K_RETURN:
‎                    choosing = False
‎        pygame.display.flip()
‎        clock.tick(FPS)
‎    return CAR_COLORS[selected]
‎
‎def main():
‎    player_color = choose_car()
‎    player_lane = 1
‎    player_y = HEIGHT - CAR_H - 20
‎    player_x = WIDTH//2 - ROAD_W//2 + player_lane*LANE_W + (LANE_W - CAR_W)//2
‎    score = 0
‎    level = 1
‎    ai_cars = [Car(random.randint(0,2), random.choice([c for c in CAR_COLORS if c != player_color]), random.randint(-600, -100)) for _ in range(5)]
‎    powerup = PowerUp()
‎    highscore = get_highscore()
‎    speed_boost = 0
‎    boost_timer = 0
‎
‎    running = True
‎    while running:
‎        screen.fill(GREEN)
‎        draw_road(level)
‎        
‎        # Player controls
‎        keys = pygame.key.get_pressed()
‎        if keys[pygame.K_LEFT] and player_lane > 0:
‎            player_lane -= 1
‎        if keys[pygame.K_RIGHT] and player_lane < 2:
‎            player_lane += 1
‎        player_x = WIDTH//2 - ROAD_W//2 + player_lane*LANE_W + (LANE_W - CAR_W)//2
‎        
‎        # Power-up
‎        powerup.update(level)
‎        powerup.draw()
‎        if powerup.active and powerup.lane == player_lane and powerup.y + CAR_H//2 > player_y and powerup.y < player_y + CAR_H:
‎            speed_boost = 10
‎            boost_timer = FPS * 2  # 2 seconds
‎            powerup.active = False
‎
‎        # AI Cars
‎        for car in ai_cars:
‎            car.update(level)
‎            car.draw()
‎            # Collision
‎            if car.lane == player_lane and car.y + CAR_H > player_y and car.y < player_y + CAR_H:
‎                running = False
‎        
‎        draw_car(player_x, player_y, player_color)
‎        
‎        score += 1 + speed_boost
‎        if score % 1000 == 0:
‎            level += 1
‎
‎        # Handle speed boost timer
‎        if boost_timer > 0:
‎            boost_timer -= 1
‎            boost_text = font.render("Speed Boost!", True, ORANGE)
‎            screen.blit(boost_text, (WIDTH//2 - 80, 10))
‎        else:
‎            speed_boost = 0
‎
‎        # Score, level, high score
‎        score_text = font.render(f"Score: {score//10}", True, WHITE)
‎        level_text = font.render(f"Level: {level}", True, YELLOW)
‎        highscore_text = font.render(f"High Score: {highscore//10}", True, PURPLE)
‎        screen.blit(score_text, (20, 20))
‎        screen.blit(level_text, (20, 60))
‎        screen.blit(highscore_text, (20, 100))
‎        
‎        for event in pygame.event.get():
‎            if event.type == pygame.QUIT:
‎                running = False
‎        
‎        pygame.display.flip()
‎        clock.tick(FPS)
‎    
‎    # Game Over
‎    over_text = font.render("Game Over!", True, RED)
‎    screen.blit(over_text, (WIDTH//2 - 100, HEIGHT//2 - 50))
‎    pygame.display.flip()
‎    pygame.time.wait(2000)
‎
‎    # Save high score
‎    if score > highscore:
‎        save_highscore(score)
‎        screen.fill(GREEN)
‎        newhigh_text = font.render("New High Score!", True, ORANGE)
‎        screen.blit(newhigh_text, (WIDTH//2 - 120, HEIGHT//2 - 50))
‎        pygame.display.flip()
‎        pygame.time.wait(1500)
‎
‎    pygame.quit()
‎    sys.exit()
‎
‎if __name__ == "__main__":
‎    main()
