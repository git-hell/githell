#!/bin/python3
import pygame
import random
pygame.init()

WIDTH = 500
HEIGHT = 500
pressed = None
screen = pygame.display.set_mode((WIDTH, HEIGHT), 0, 32)
pygame.display.set_caption('Santa Game')
font = pygame.font.SysFont('Lato', 24)

santa = pygame.image.load('santa.png')
SANTA_WIDTH, SANTA_HEIGHT = santa.get_size()
santa = pygame.transform.scale(santa, (SANTA_WIDTH // 2, SANTA_HEIGHT // 2))
score = 0
santa_y = HEIGHT / 2

gift_images = [pygame.image.load('gift1.png'), pygame.image.load('gift2.png')] * 10
GIFT_WIDTH, GIFT_HEIGHT = gift_images[0].get_size()
gift_images = [pygame.transform.scale(gift, (GIFT_WIDTH // 2, GIFT_HEIGHT // 2)) for gift in gift_images]
GIFT_SPEED = 0.3
gift_index = 0
gifts = []
uped = False
for i in range(5):
    gifts.append([[WIDTH + GIFT_WIDTH, random.randint(0, HEIGHT)], random.choice(gift_images), GIFT_SPEED])

snowflakes = []
for i in range(30):
    snowflakes.append([[random.randint(0, WIDTH), random.randint(0, HEIGHT)], random.randint(2, 8)])

while True:
    screen.fill((0, 0, 0))
    i = 0
    for snowflake in snowflakes:
        pygame.draw.circle(screen, (255, 255, 255), snowflake[0], snowflake[1])
        snowflake[0][1] += 1
        if snowflake[0][1] > HEIGHT + snowflake[1] * 2:
            snowflake[0][1] = 0 - snowflake[1] * 2
        snowflakes[i] = snowflake
        i += 1

    screen.blit(santa, (0, santa_y))

    gift = gifts[gift_index]
    screen.blit(gift[1], gift[0])
    gift[0][0] -= gift[2]
    if gift[0][0] < 0 - GIFT_WIDTH / 2:
        score -= 1
        gift[0][0] = WIDTH + GIFT_WIDTH
        gift[0][1] = random.randint(0, HEIGHT)
        if gift_index < len(gifts) - 1:
            gift_index += 1
        else:
            gift_index = 0
    gift[2] = GIFT_SPEED
    gifts[gift_index] = gift

    santa_rect = santa.get_rect()
    santa_rect.x = 0
    santa_rect.y = santa_y

    gift_rect = gift[1].get_rect()
    gift_rect.x = gift[0][0]
    gift_rect.y = gift[0][1]

    gift[2] = GIFT_SPEED

    if santa_rect.colliderect(gift_rect):
        score += 1
        uped = False
        if not uped:
            GIFT_SPEED += 0.1
            uped = True
        gift[0][0] = WIDTH + GIFT_WIDTH
        gift[0][1] = random.randint(0, HEIGHT)
        if gift_index < len(gifts) - 1:
            gift_index += 1
        else:
            gift_index = 0
        gifts[gift_index] = gift
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
        elif event.type == pygame.KEYDOWN:
            pressed = event.key
        elif event.type == pygame.KEYUP:
            pressed = None
    if pressed == pygame.K_UP:
        if santa_y > 0:
            santa_y -= 1
    elif pressed == pygame.K_DOWN:
        if santa_y < HEIGHT - SANTA_HEIGHT / 2:
            santa_y += 1

    score_text = font.render('Score: ' + str(score), False, (255, 255, 255))
    screen.blit(score_text, (WIDTH / 2, 0))

    pygame.display.flip()
