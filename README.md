# Final-game-for-the-year
import pygame
import sys
pygame.init()
screen = pygame.display.set_mode((800, 600))
pygame.display.set_caption("super!")
#=background_image = pygame.image.load('background.png').convert_alpha()
playerani = pygame.image.load('knightfinal.png').convert_alpha()
clock = pygame.time.Clock()
GRAVITY = 0.5
keys = pygame.key.get_pressed()
# Colors
WHITE = (255, 255, 255)
BLUE = (0, 100, 255)
GREEN = (0, 200, 0)
#variables
offset = 0
bg_x1 = 0
bg_x2 = 800

ticker = 0

total_time = 60
font = pygame.font.SysFont('freesansbold.ttf', 40)  
score = 0

xpos = 200 #holding the mouse position variables
ypos = 200
mousePos = (xpos, ypos)
shop_open = False

lives = 3

player_start = (100, 100)


#-class platform----------------------------------------------------------------------------------------------
class Platform:
   
    def __init__(self, x, y, w, h): #constructor
        global offset
        self.x = x
        self.y = y
        self.w = w
        self.h = h
        self.image = pygame.image.load('platformimg.png').convert_alpha()

    def draw(self, surface): #draw function
        pygame.draw.rect(surface, GREEN, (self.x+offset, self.y, self.w, self.h))
        #screen.blit(self.image, (self.x + offset, self.y))
       

#-class player----------------------------------------------------------------------------------------------
class Player:
   
    def __init__(self, x, y): #constructor
        self.x = x
        self.y = y
        self.w = 40
        self.h = 60
        self.vy = 0
        self.vx = 0
        self.on_ground = False
        self.frameWidth = 45
        self.frameHeight = 65
        self.Rows = 0
        self.frameNum = 0

           
    def handle_input(self, keys): #keyboard input
        global offset
        if keys[pygame.K_RIGHT]:
            #self.x += 5
            #self.xv = 0
            offset -= 5
            self.Rows = 0
            self.frameNum +=1
            if self.frameNum > 2:
                self.frameNum = 0
        elif keys[pygame.K_LEFT]:
            #self.x -= 5
            #self.vx = 0
            offset +=5
            self.Rows = 1
            self.frameNum += 1
            if self.frameNum > 2:
                self.frameNum = 0
        if keys[pygame.K_UP] and self.on_ground:
            self.vy = -12
            self.on_ground = False

       
           

    def apply_gravity(self): #make player fall
        self.vy += GRAVITY
        self.y += self.vy

    def check_collision(self, platforms):
        self.on_ground = False
        for plat in platforms:
            plat_x = plat.x + offset

            if self.x + self.frameWidth > plat.x + offset and self.x < plat.x + offset + plat.w:
                    if self.y + self.frameHeight > plat.y and self.y + self.frameHeight - self.vy <= plat.y:
                        self.y = plat.y - self.frameHeight
                        self.vy = 0
                        self.on_ground = True  
           
            # Check vertical collision
            if self.x + self.w > plat_x and self.x < plat_x + plat.w:
                if self.y + self.h > plat.y and self.y + self.h - self.vy <= plat.y:
                    self.y = plat.y - self.h
                    self.vy = 0
                    self.on_ground = True

            # Check horizontal collision
            if self.y + self.h > plat.y and self.y < plat.y + plat.h:
                if self.x + self.w > plat_x and self.x < plat_x:
                    self.x = plat_x - self.w
                elif self.x < plat_x + plat.w and self.x + self.w > plat_x + plat.w:
                    self.x = plat_x + plat.w

            if self.y >= 580:
                sys.exit()

            for plat in movingplatforms:
                if self.on_ground and self.y + self.h == plat.y and self.x +offset + self.w > plat.x + offset and self.x < plat.x + offset + plat.w:
                    self.x += plat.speed * plat.direction

    def is_colliding(self, plat): #bounding box collision
        global offset
        #print("offset is", offset)
        if self.x + self.w > plat.x+offset and self.x < plat.x+offset + plat.w and self.y + self.h > plat.y and self.y < plat.y + plat.h:
            print("colliding")
            
        
    def checkpoint_colliding(self, xpos,ypos,width,height,num): #bounding box collision
        global offset
        #print("offset is", offset)
        if self.x + self.w > plat.x+offset and self.x < plat.x+offset + plat.w and self.y + self.h > plat.y and self.y < plat.y + plat.h:
            #print("colliding")
            return True

       

    def update(self, platforms, movingplatforms): #funtion that calls a bunch of other functions (keeps game loop more simple)
        self.apply_gravity()
        self.check_collision(platforms + movingplatforms)
        self.x += self.vx




    def draw(self, surface):
        surface.blit(playerani, (self.x, self.y), (self.frameWidth * self.frameNum, self.Rows * self.frameHeight, self.frameWidth, self.frameHeight))

class Spikes:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.image = pygame.image.load('spike.png').convert_alpha()
        self.image = pygame.transform.scale(self.image, (25, 25))
       
       
    def draw(self, surface):
        global offset
        #pygame.draw.polygon(surface, (0, 0, 200), [(self.x+offset+50, self.y+12.5), (self.x+37.5+offset, self.y-12.5), (self.x+25+offset, self.y+12.5)])
        screen.blit(self.image, (self.x + offset, self.y))

    def is_colliding(self, player):
        global offset
        if (player.x + player.w > self.x + offset and
            player.x < self.x + offset + 10.5 and  
            player.y + player.h > self.y - 5.5 and  
            player.y < self.y + 5.5):  
            return True
        return False
   
class Projectile:
    def __init__(self, x, y, speed):
        self.x = x
        self.y = y
        self.speed = speed
        self.w = 10
        self.h = 10
        self.color = (255, 0, 0)
        self.image = pygame.image.load('cannonproj.png').convert_alpha()

    def update(self):
        self.x -= self.speed  

    def draw(self, surface):
        #pygame.draw.rect(surface, self.color, (self.x + offset, self.y, self.w, self.h))
        screen.blit(self.image, (self.x + offset, self.y, self.w, self.h))

    def is_colliding(self, player):
        if (player.x + player.w > self.x + offset and
            player.x < self.x + offset + self.w and
            player.y + player.h > self.y and
            player.y < self.y + self.h):
            return True
        return False
   
class Cannon:
    def __init__(self, x, y, w, h):
        self.x = x
        self.y = y
        self.w = w
        self.h = h
        self.cooldown = 0  # Timer for shooting
        self.projectiles = []
        self.image = pygame.image.load('cannon.png').convert_alpha()
       
    def draw(self, surface):
        #pygame.draw.rect(surface, (0, 0, 0), (self.x + offset, self.y, self.w, self.h))
        screen.blit(self.image, (self.x + offset, self.y, self.w, self.h))
        for projectile in self.projectiles:
            projectile.draw(surface)

    def shoot(self):
        if self.cooldown == 0:
            self.projectiles.append(Projectile(self.x, self.y + self.h // 2, 5))
            self.cooldown = 80
       
        if self.cooldown > 0:
            self.cooldown -= 1

    def update(self):
        for projectile in self.projectiles:
            projectile.update()
            if projectile.x < -50:  # Remove off-screen projectiles
                self.projectiles.remove(projectile)

class Coin:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.image = pygame.image.load('coin.png').convert_alpha()
        self.image = pygame.transform.scale(self.image, (40, 40))

    def draw(self, surface):
        screen.blit(self.image, (self.x + offset, self.y))

    def playerCoincollision(self, player):
     global score  
     if (player.x + player.w > self.x + offset and
         player.x < self.x + offset + 40 and  
         player.y + player.h > self.y and
         player.y < self.y + 40):  
         score += 10
         return True  
     return False
   
class Shop:
    def __init__(self, x, y, w, h):
        self.x = x
        self.y = y
        self.w = w
        self.h = h
 
    def draw(self, surface):
        global offset
        pygame.draw.rect(surface, (0, 0, 0), (self.x + offset, self.y, self.w, self.h))

    def playerShopcollision(self, player):
        global shop_open
        if (player.x + player.w > self.x + offset and
         player.x < self.x + offset + 40 and  
         player.y + player.h > self.y and
         player.y < self.y + 40):
            if keys[pygame.K_LSHIFT]:
                print("test")
                shop_open = True

    def draw_shop_screen(self, surface):
        pygame.draw.rect(surface, (200, 200, 200), (100, 100, 600, 400))
        pygame.draw.rect(surface, (0, 0, 0), (100, 100, 600, 400), 5)
        text = font.render("Shop - Press ESC to Exit", True, (0, 0, 0))
        surface.blit(text, (120, 120))

class Enemy:
    def __init__(self, x, y, w, h, min_x, max_x, speed=2):
        self.x = x
        self.y = y
        self.w = w
        self.h = h
        self.min_x = min_x
        self.max_x = max_x
        self.speed = speed
        self.direction = 1
        self.image = pygame.image.load('ghost.png').convert_alpha()
        self.image = pygame.transform.scale(self.image, (35, 35))

    def update(self):
        self.x += self.speed * self.direction
        if self.x < self.min_x or self.x + self.w > self.max_x:
            self.direction *= -1

    def draw(self, surface):
        screen.blit(self.image, (self.x + offset, self.y))

    def is_colliding(self, player):
        if (player.x + player.w > self.x + offset and
         player.x < self.x + offset + 40 and  
         player.y + player.h > self.y and
         player.y < self.y + 40):
            print("test")
           

class Checkpoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.activated = False

    def draw(self, surface):
        pygame.draw.rect(surface, (0, 0, 0), (self.x + offset, self.y, 20, 50))

    def is_colliding(self, player):
        if (player.x + player.w > self.x + offset and
            player.x < self.x + offset + 20 and  
            player.y + player.h > self.y and
            player.y < self.y + 50):
            self.activated = True
            return True
        else:
            return False
           
class MovingPlatform:
     def __init__(self, x, y, w, h, min_x, max_x, speed=4):
        self.x = x
        self.y = y
        self.w = w
        self.h = h
        self.min_x = min_x
        self.max_x = max_x
        self.speed = speed
        self.direction = 1
   
     def update(self):
            self.x += self.speed * self.direction
            if self.x < self.min_x or self.x + self.w > self.max_x:
                self.direction *= -1

     def draw(self, surface):
         pygame.draw.rect(surface, GREEN, (self.x + offset, self.y, self.w, self.h))

     def is_colliding(self, player):
        if (player.x + player.w > self.x + offset and
         player.x < self.x + offset + 40 and  
         player.y + player.h > self.y and
         player.y < self.y + 40):
            reset_game() == (self.x, self.y)

def reset_game():
    global player, offset  # Make sure to reset the player and offset
    player.x = 100  # Reset player to starting x position
    player.y = 100  # Reset player to starting y position
    offset = 0  
#list to contain platforms
platforms = [
    Platform(100, 500, 600, 20), #calling platform class constructor
    Platform(300, 400, 100, 100),
    Platform(600, 300, 100, 200),
    Platform(845, 245, 100, 20),
    Platform(1100, 400, 100, 20),
    Platform(1400, 400, 100, 20),
    Platform(2000, 380, 100, 20),
    Platform(1700, 380, 100, 20),
    Platform(2200, 380, 500, 20),
    Platform(3500, 380, 100, 20)

]

spikes = [
    Spikes(400, 475),
    Spikes(425, 475),
    Spikes(450, 475),
    Spikes(475, 475),
    Spikes(500, 475),
    Spikes(525, 475),
    Spikes(550, 475),
    Spikes(575, 475),
    Spikes(845, 220),
    Spikes(920, 220),
    Spikes(2700, 355)
   
]

cannons = [
    Cannon(2000, 350, 50, 30),
    Cannon(3500, 350, 50, 30)
]

coins = [
    Coin(330, 360)
]
player = Player(100, 100) #calling player class constructor

shops = [
    Shop(2200, 350, 100, 100)

]

enemies = [
    Enemy(2250, 355, 25, 25, 2250, 2350),
    Enemy(2350, 355, 25, 25, 2350, 2450)
]

checkpoints = [
    Checkpoint(2600, 330)
]

movingplatforms = [
    MovingPlatform(2800, 380, 100, 20, 2800, 3450),
   

]
attacks = []
   
if keys[pygame.K_RIGHT]:
        if offset < -1500 and player[0]<750:
            player.vx = 5
       
        elif offset >260 and player[0]<400:
            player.vx = 5
       
        elif player.x<750:
            offset -= 5
            player.vx = 0
       
        else:
            player.vx = 0
if keys[pygame.K_LEFT]:
        if offset > 260 and player[0]>0: #check if youve reached the left edge of the map
            player.vx = -5 #let player approach side of game screen
       
        elif player.vy>400 and offset < -1500:#check if were on the far right edge of the map
            player.vx = -5 #let player get back to the center of the game screen
           
        elif player.x>0: #if player is recenbtered, move the *offset*, not the player
            offset += 5
            player.vx = 0
           
        else:
            player.vx = 0 #make sure motion is off (stops from going off edge.
   

start_ticks = pygame.time.get_ticks()
running = True
while running: #GAME LOOP############################################################################
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            over = True
    if event.type == pygame.MOUSEMOTION:
            mousePos = event.pos
            print("mouse position: (",mousePos[0], " , ",mousePos[1], ")")
    #input section-------------------
    clock.tick(60)
    print(player_start)
   
    seconds = total_time - (pygame.time.get_ticks() - start_ticks) // 1000
    if seconds <= 0:
        print("Time's up!")
        running = False
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    player.handle_input(keys)

    for shop in shops:
        if shop.playerShopcollision(player) and keys[pygame.K_LSHIFT]:
            shop_open = True  # Open the shop
            print("Shop opened")
    for checkpoint in checkpoints:
        if checkpoint.is_colliding(player):
            player_start = checkpoint
           
   

    for attack in attacks[:]:
        attack.update()
        if not attack.active:
            attacks.remove(attack)


    for enemy in enemies:
        enemy.update()

    if shop_open:
        shop.draw_shop_screen(screen)
        if keys[pygame.K_ESCAPE]:
            shop_open = False



           
    #update/physics section-----------
    player.update(platforms, movingplatforms)

    for movingplatform in movingplatforms:
        movingplatform.update()
    for coin in coins[:]:  
        if coin.playerCoincollision(player):
            coins.remove(coin)
    #render section-------------------
    screen.fill(WHITE)
    #screen.blit(background_image, (0, 0))
    timer_text = font.render(f"Time: {seconds}", True, BLUE)
    screen.blit(timer_text, (620, 10))

    playerscore = font.render(f"score: {score}", True, BLUE)
    screen.blit(playerscore, (5, 10))

    lives_text = font.render(f"Lives: {lives}", True, BLUE)
    screen.blit(lives_text, (5, 50))
    for plat in platforms:
        plat.draw(screen)

    for spike in spikes:
        spike.draw(screen)
   
    for cannon in cannons:
        cannon.draw(screen)

    for enemy in enemies:
        enemy.draw(screen)

    for movingplatform in movingplatforms:
        movingplatform.draw(screen)

   # for shop in shops:
       # shop.draw(screen)



    for checkpoint in checkpoints:
        checkpoint.draw(screen)

    for shop in shops:
        if shop.playerShopcollision(player) and keys[pygame.K_LSHIFT]:
            shop_open = True  # Open the shop
            print("Shop opened")

    if shop_open and keys[pygame.K_ESCAPE]:
        shop_open = False

    for cannon in cannons:
        screen_x = cannon.x + offset
        if 0 <= screen_x <= 800:
            cannon.shoot()
        cannon.update()
        for projectile in cannon.projectiles:
            if projectile.is_colliding(player):
                lives -= 1
                reset_game()  # Respawn player at starting position
                if lives <= 0:
                    running = False

    for enemy in enemies:
        if enemy.is_colliding(player):
            lives -= 1
            reset_game()  # Respawn player at starting position
            if lives <= 0:
                running = False

    for spike in spikes:
        if spike.is_colliding(player):
            lives -= 1
            reset_game()  # Respawn player at starting position
            if lives <= 0:
                running = False

    for coin in coins:
        coin.draw(screen)
    player.draw(screen)

    for attack in attacks:
        attack.draw(screen)



    pygame.display.flip()
   
#END OF GAME LOOP############################################################################

pygame.quit()
