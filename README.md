# Final-game-for-the-year

import pygame


pygame.init()
screen = pygame.display.set_mode((800, 600))
pygame.display.set_caption("super!")
background_image = pygame.image.load('placeholderback.png').convert_alpha()
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
frameWidth = 64
frameHeight = 96
RowNum = 0
frameNum = 0
ticker = 0
#-class platform----------------------------------------------------------------------------------------------
class Platform:
    
    def __init__(self, x, y, w, h): #constructor
        self.x = x
        self.y = y
        self.w = w
        self.h = h

    def draw(self, surface): #draw function
        pygame.draw.rect(surface, GREEN, (self.x+offset, self.y, self.w, self.h))
        

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

    def handle_input(self, keys): #keyboard input
        global offset
        if keys[pygame.K_RIGHT]:
            #self.x += 5
            #self.xv = 0 
            offset -= 5
        if keys[pygame.K_LEFT]:
            #self.x -= 5
            #self.vx = 0
            offset +=5
        if keys[pygame.K_UP] and self.on_ground:
            self.vy = -12
            self.on_ground = False
            

    def apply_gravity(self): #make player fall
        self.vy += GRAVITY
        self.y += self.vy

    def check_collision(self, platforms): 
        self.on_ground = False #assume we're in the air, change if not
        for plat in platforms: #check all the platforms in the list
            if self.is_colliding(plat): #if we ARE colliding, reset feet to top of platform...
                if self.y + self.h <= plat.y + self.vy:
                    self.y = plat.y - self.h
                    self.vy = 0
                    self.on_ground = True

    def is_colliding(self, plat): #bounding box collision
        global offset
        #print("offset is", offset)
        if self.x + self.w > plat.x+offset and self.x < plat.x+offset + plat.w and self.y + self.h > plat.y and self.y < plat.y + plat.h:
            #print("colliding")
            return True
        else:
            return False
        

    def update(self, platforms): #funtion that calls a bunch of other functions (keeps game loop more simple)
        self.apply_gravity()
        self.check_collision(platforms)

    def draw(self, surface):
        pygame.draw.rect(surface, (255, 0, 0), (self.x, self.y, self.w, self.h))

#-end of classes----------------------------------------------------------------------------------------------

#list to contain platforms
platforms = [
    Platform(100, 500, 1300, 20), #calling platform class constructor
    Platform(300, 400, 100, 10),
    Platform(500, 300, 100, 10)
]

player = Player(100, 100) #calling player class constructor


    
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
    


running = True
while running: #GAME LOOP############################################################################
    
    #input section-------------------
    clock.tick(60)
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    player.handle_input(keys)
    
    #update/physics section-----------
    player.update(platforms)

    #render section-------------------
    screen.fill(WHITE)
    screen.blit(background_image, (0, 0))



    if player.vy < 0:
        ticker+=1
        if ticker%10==0:
            frameNum+=1
            
        if frameNum>7:
            frameNum = 0
   # bg_x1 -= 2
   # bg_x2 -= 2

  #  if bg_x1 <= -800:
   #     bg_x1 = 800
   # if bg_x2 <= -800:
    #    bg_x2 = 800

    #screen.blit(background_image, (bg_x1, 0))
    #screen.blit(background_image, (bg_x2, 0))
    for plat in platforms:
        plat.draw(screen)

    player.draw(screen)




    pygame.display.flip()
    
#END OF GAME LOOP############################################################################

pygame.quit()
