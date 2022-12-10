# Orc_Raid
import pygame
import random
import os
import sys
import time

# ----- 게임창 위치설정 -----
win_posx = 400
win_posy = 50
os.environ['SDL_VIDEO_WINDOW_POS'] = "%d,%d" % (win_posx, win_posy)

# ----- 전역 -----
SCREEN_WIDTH = 400
SCREEN_HEIGHT = 600
FPS = 60

score = 0
playtime = 1

# ----- 색상 -----
BLACK = 0, 0, 0
WHITE = 255,255,255
RED = 255, 0, 0
GREEN1 = 25, 102, 25
GREEN2 = 51, 204, 51
GREEN3 = 233, 249, 185
BLUE = 17, 17, 212
YELLOW = 255, 255, 0

def initialize_game(width, height):
    pygame.init()
    surface = pygame.display.set_mode((width, height))
    pygame.display.set_caption("Orc Raid")
    return surface

def menu_loop(surface):
    background = pygame.image.load(os.path.join("C:/image/OrcRaid.png")).convert()
    background = pygame.transform.scale(background, (SCREEN_WIDTH, SCREEN_HEIGHT))
    bg_rect = background.get_rect()

    clock = pygame.time.Clock()

    running = True
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                 running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_q:
                    running = False
                if event.key == pygame.K_SPACE:
                    game_loop(surface)

        surface.blit(background, bg_rect)
        menu_show(surface)
        pygame.display.flip()
        clock.tick(FPS)
        
    pygame.quit()

def menu_show(surface):
    f = open("C:/image/score.txt", 'r')
    firstline = f.readline() # 첫번째 줄 읽어옴
    file_score = firstline
    best_score = int(file_score)
    f.close()
    
    font = pygame.font.SysFont('malagungothic',35)
    image = font.render(f'  Press SPACE to start    ', True, BLACK)
    image2 = font.render(f'  Press Q to quit    ', True, BLACK)
    image_Bscore = font.render(f'  Best Score :  {best_score}   ', True, BLACK)
    pos = image.get_rect()
    pos.move_ip(35 ,SCREEN_HEIGHT/2+50)
    pos2 = image2.get_rect()
    pos2.move_ip(35 ,SCREEN_HEIGHT/2+100)
    pos3 = image_Bscore.get_rect() 
    pos3.move_ip(35 ,SCREEN_HEIGHT/2+150) 
    surface.blit(image, pos)
    surface.blit(image2, pos2)
    surface.blit(image_Bscore, pos3)
    pygame.display.update()

def game_loop(surface):
    background = pygame.image.load(os.path.join("C:/image/fightmap.png")).convert()
    background = pygame.transform.scale(background, (SCREEN_WIDTH, SCREEN_HEIGHT))
    bg_rect = background.get_rect()
    player_image = pygame.image.load(os.path.join("C:/image/Bowman.png")).convert()
    asteroid_image = []
    asteroid_list = ['orc1.png', 'orc2.png', 'orc3.png', 'orc4.png']
    for img in asteroid_list:
        asteroid_image.append(pygame.image.load(os.path.join("C:/image", img)).convert())
    bullet_image = pygame.image.load(os.path.join("C:/image/Bullet.png")).convert()
    #bullet_image = pygame.transform.scale(bullet_image, (30, 30))

    clock = pygame.time.Clock()
    sprite_group = pygame.sprite.Group()
    mobs = pygame.sprite.Group()
    bullets = pygame.sprite.Group()
    n_bullets = pygame.sprite.Group()
    nw_bullets = pygame.sprite.Group()
    w_bullets = pygame.sprite.Group()
    sw_bullets = pygame.sprite.Group()
    s_bullets = pygame.sprite.Group()
    se_bullets = pygame.sprite.Group()
    e_bullets = pygame.sprite.Group()
    ne_bullets = pygame.sprite.Group()
    player = PlayerShip(player_image)

    global player_health
    player_health= 100
    global score
    score = 0
    global gage
    gage = 0
    
    sprite_group.add(player)
    for i in range(10):
        enemy = Mob(asteroid_image)
        sprite_group.add(enemy)
        mobs.add(enemy)

    running = True
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                 running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    pause_loop(surface)
                if event.key == pygame.K_SPACE:
                    player.teleport()
                if event.key == pygame.K_r:
                    if gage >= 100:
                        player.shoot_n(sprite_group, n_bullets, bullet_image)
                        player.shoot_nw(sprite_group, nw_bullets, bullet_image)
                        player.shoot_w(sprite_group, w_bullets, bullet_image)
                        player.shoot_sw(sprite_group, sw_bullets, bullet_image)
                        player.shoot_s(sprite_group, s_bullets, bullet_image)
                        player.shoot_se(sprite_group, se_bullets, bullet_image)
                        player.shoot_e(sprite_group, e_bullets, bullet_image)
                        player.shoot_ne(sprite_group, ne_bullets, bullet_image)
                        gage -= 100
                    else:
                        print("low gage!")                        
                        #print message
            if event.type == pygame.MOUSEBUTTONDOWN:
                player.shoot(sprite_group, bullets, bullet_image)

        sprite_group.update()

        hits = pygame.sprite.spritecollide(player, mobs, False) #True => dead, False =>live
        if hits:
            print('a mob hits player!')            
            player_health -= 5
            if player_health <= 0:
                score_update(surface)
                gameover(surface)                
                close_game()
                restart()

        hits = pygame.sprite.groupcollide(mobs, bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10
            if gage >= 100:
                gage = 100
            else:
                gage += 10

        hits = pygame.sprite.groupcollide(mobs, n_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10

        hits = pygame.sprite.groupcollide(mobs, nw_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10

        hits = pygame.sprite.groupcollide(mobs, w_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10

        hits = pygame.sprite.groupcollide(mobs, sw_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10

        hits = pygame.sprite.groupcollide(mobs, s_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10

        hits = pygame.sprite.groupcollide(mobs, se_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10

        hits = pygame.sprite.groupcollide(mobs, e_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10

        hits = pygame.sprite.groupcollide(mobs, ne_bullets, True, True)
        for hit in hits:
            mob = Mob(asteroid_image)
            sprite_group.add(mobs)
            mobs.add(mob)
            score += 10
                
        surface.blit(background, bg_rect)
        sprite_group.draw(surface)
        score_update(surface)
        if gage >= 100:
                gage_show(surface)
        pygame.display.flip()
        clock.tick(FPS)
    pygame.quit()
    update_best_score()
    print('game played: ',playtime)

def update_best_score():
    f = open("C:/image/score.txt", 'r')
    firstline = f.readline()
    file_score = firstline
    best_score = int(file_score)
    f.close()

    if(best_score < score):
        best_score = score
        best_score = str(best_score)
        f = open('C:/image/score.txt', 'w')
        f.truncate()
        f.write(best_score)
           
    print(best_score)
    f.close()    

def pause_loop(surface):

    clock = pygame.time.Clock()    

    paused = True
    while paused:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_m:
                    update_best_score()
                    menu_loop(screen)
                if event.key == pygame.K_q:
                    update_best_score()              
                    pygame.quit()
                    quit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                paused = False

        pause_show(surface)
        pygame.display.flip()
        clock.tick(FPS)    

def score_update(surface):
    font = pygame.font.SysFont('malagungothic',30)
    font2 = pygame.font.SysFont('malagungothic',25)
    image = font.render(f'  SCORE : {score}  HP : {player_health}   GAGE : {gage} ', True, BLACK)
    image2 = font2.render(f'  Press ESC to pause ', True, BLACK)
    pos = image.get_rect()
    pos2 = image2.get_rect()
    pos.move_ip(20,20)
    pos2.move_ip(20,50)    
    surface.blit(image, pos)
    surface.blit(image2, pos2)

def gage_show(surface):
    font = pygame.font.SysFont('malgungothic',20)
    image = font.render(f'  Press R! ', True, RED)
    pos = image.get_rect()
    pos.move_ip(20, 80)
    surface.blit(image, pos)

def pause_show(surface): 
    font = pygame.font.SysFont('malgungothic',20)
    image = font.render(f'  Press MOUSE to continue ', True, BLACK)
    image2 = font.render(f'  Press m to go MENU ', True, BLACK)
    image3 = font.render(f'  Press Q to quit ', True, BLACK)
    pos = image.get_rect()
    pos2 = image.get_rect()
    pos3 = image.get_rect()
    pos.move_ip(50, int(SCREEN_HEIGHT/2-50))
    pos2.move_ip(50, int(SCREEN_HEIGHT/2+50))
    pos3.move_ip(50, int(SCREEN_HEIGHT/2+150))    
    surface.blit(image, pos)
    surface.blit(image2, pos2)
    surface.blit(image3, pos3)
    pygame.display.update()     

def gameover(surface):
    font = pygame.font.SysFont('malgungothic',50)
    image = font.render('GAME OVER', True, BLACK)
    pos = image.get_rect()
    pos.move_ip(50, int(SCREEN_HEIGHT/2))
    surface.blit(image, pos)
    pygame.display.update()
    time.sleep(2)

def close_game():
    pygame.quit()
    print('Game closed')

def restart():
    screen = initialize_game(SCREEN_WIDTH,SCREEN_HEIGHT)
    game_loop(screen)
    close_game()

class PlayerShip(pygame.sprite.Sprite):
    def __init__(self, image):
        pygame.sprite.Sprite.__init__(self)
        #self.image = pygame.Surface((30, 30))
        self.image = pygame.transform.scale(image, (30, 30))
        #self.image.fill(BLACK)
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.centerx = int(SCREEN_WIDTH / 2)
        self.rect.centery = SCREEN_HEIGHT - 20
        self.speedx = 0
        self.speedy = 0

    def update(self):
        self.speedx = 0
        self.speedy = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_a]:
            self.speedx = -3
        if keystate[pygame.K_d]:
            self.speedx = 3
        if keystate[pygame.K_w]:
            self.speedy = -3
        if keystate[pygame.K_s]:
            self.speedy = 3
        self.rect.x += self.speedx
        self.rect.y += self.speedy
        if self.rect.right > SCREEN_WIDTH:
            self.rect.right = SCREEN_WIDTH
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.bottom > SCREEN_HEIGHT:
            self.rect.bottom = SCREEN_HEIGHT
        if self.rect.top < 0:
            self.rect.top = 0

    def teleport(self):
        self.speedx = 0
        self.speedy = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_SPACE]:            
            (self.rect.x, self.rect.y) = pygame.mouse.get_pos()

    def shoot(self, all_sprites, bullets, image):
        bullet = Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(bullet)
        bullets.add(bullet)

    def shoot_n(self, all_sprites, bullets, image):
        bullet = n_Bullet(self.rect.centerx, self.rect.top, image)        
        all_sprites.add(bullet)
        bullets.add(bullet)
         
    def shoot_nw(self, all_sprites, bullets, image):
        nw_bullet = nw_Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(nw_bullet)
        bullets.add(nw_bullet)

    def shoot_w(self, all_sprites, bullets, image):
        w_bullet = w_Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(w_bullet)
        bullets.add(w_bullet)

    def shoot_sw(self, all_sprites, bullets, image):
        bullet = sw_Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(bullet)
        bullets.add(bullet)

    def shoot_s(self, all_sprites, bullets, image):
        bullet = s_Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(bullet)
        bullets.add(bullet)

    def shoot_se(self, all_sprites, bullets, image):
        bullet = se_Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(bullet)
        bullets.add(bullet)

    def shoot_e(self, all_sprites, bullets, image):
        bullet = e_Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(bullet)
        bullets.add(bullet)

    def shoot_ne(self, all_sprites, bullets, image):
        bullet = ne_Bullet(self.rect.centerx, self.rect.top, image)
        all_sprites.add(bullet)
        bullets.add(bullet)
    
class Mob(pygame.sprite.Sprite):
    def __init__(self, image):
        pygame.sprite.Sprite.__init__(self)
        self.image_origin = random.choice(image)
        self.image_origin = pygame.transform.rotozoom(random.choice(image), 0, 0.7)
        self.image = self.image_origin
        self.image.set_colorkey(BLACK)
        #self.image = pygame.Surface((30,30))
        #self.color = random.choice([BLUE, RED, GREEN1, YELLOW])
        #self.image.fill(self.color)
        self.rect = self.image.get_rect()

        self.rect.x = random.randrange(SCREEN_WIDTH - self.rect.width)
        self.rect.y = random.randrange(-100, -99)
        self.speedy = random.randrange(3, 5)
        self.speedx = random.randrange(-3, 3)
        self.direction_change = True

    def update(self):
        self.rect.x += self.speedx
        self.rect.y += self.speedy

        if self.rect.top > SCREEN_HEIGHT + 10 or self.rect.left < -25 or self.rect.right > SCREEN_WIDTH + 20:
            self.rect.x = random.randrange(SCREEN_WIDTH - self.rect.width)
            self.rect.y = random.randrange(-100, -40)
            self.speedy = random.randrange(3, 8)

class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)
        #self.image = pygame.Surface((5,5))
        #self.image.fill(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = -10

    def update(self):
        self.rect.y += self.speedy
        if self.rect.bottom < 0:
            self.kill()

class n_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):        
            pygame.sprite.Sprite.__init__(self)
            self.image = pygame.transform.scale(image, (20, 20))
            self.image.set_colorkey(BLACK)
            self.rect = self.image.get_rect()
            self.rect.bottom = y
            self.rect.centerx = x
            self.speedy = -10

    def update(self):         
        self.rect.y += self.speedy        
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

class nw_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = -5
        self.speedx = -5

    def update(self):
        self.rect.x += self.speedx 
        self.rect.y += self.speedy        
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

class w_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x                
        self.speedx = -10

    def update(self):
        self.rect.x += self.speedx               
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

class sw_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)        
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = 5
        self.speedx = -5

    def update(self):
        self.rect.x += self.speedx 
        self.rect.y += self.speedy        
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

class s_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = 10

    def update(self):        
        self.rect.y += self.speedy        
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

class se_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = 5
        self.speedx = 5

    def update(self):
        self.rect.x += self.speedx 
        self.rect.y += self.speedy        
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

class e_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedx = 10

    def update(self):
        self.rect.x += self.speedx               
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

class ne_Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(image, (20, 20))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = -5
        self.speedx = 5

    def update(self):
        self.rect.x += self.speedx 
        self.rect.y += self.speedy        
        if self.rect.bottom < 0 or self.rect.bottom > SCREEN_HEIGHT:
            self.kill()       
        if self.rect.x < 0 or self.rect.x > SCREEN_WIDTH :
            self.kill()

if __name__ == '__main__':
    screen = initialize_game(SCREEN_WIDTH,SCREEN_HEIGHT)
    menu_loop(screen)    
    sys.exit()
