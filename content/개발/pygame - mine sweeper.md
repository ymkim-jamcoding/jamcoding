---
description:
aliases:
created: 2026-04-04
modified: 2026-04-04
---

```python
import sys
from math import floor
from random import randint

import pygame
from pygame import QUIT, MOUSEBUTTONDOWN, KEYDOWN

WIDTH = 20
HEIGHT = 15
SIZE = 50
NUM_OF_BOMBS = 20
EMPTY = 0
BOMB = 1
OPENED = 2

OPEN_COUNT = 0
CHECKED = [[False for _ in range(WIDTH)] for _ in range(HEIGHT)]

pygame.init()
SURFACE = pygame.display.set_mode([WIDTH * SIZE, HEIGHT * SIZE])
FPSCLOCK = pygame.time.Clock()


def init_game():
    global OPEN_COUNT, CHECKED

    OPEN_COUNT = 0
    CHECKED = [[False for _ in range(WIDTH)] for _ in range(HEIGHT)]

    field = [[EMPTY for _ in range(WIDTH)] for _ in range(HEIGHT)]

    count = 0
    while count < NUM_OF_BOMBS:
        xpos, ypos = randint(0, WIDTH - 1), randint(0, HEIGHT - 1)
        if field[ypos][xpos] == EMPTY:
            field[ypos][xpos] = BOMB
            count += 1

    return field


def num_of_bomb(field, x_pos, y_pos):
    result = 0
    for yoffset in range(-1, 2):
        for xoffset in range(-1, 2):
            xpos, ypos = (x_pos + xoffset, y_pos + yoffset)
            if xpos < 0 or WIDTH <= xpos:
                continue
            if ypos < 0 or HEIGHT <= ypos:
                continue
            if field[ypos][xpos] != BOMB:
                continue

            result += 1

    return result


def open_tile(field, x_pos, y_pos):
    global OPEN_COUNT

    if CHECKED[y_pos][x_pos]:
        return

    CHECKED[y_pos][x_pos] = True

    for yoffset in range(-1, 2):
        for xoffset in range(-1, 2):
            xpos, ypos = (x_pos + xoffset, y_pos + yoffset)

            if xpos < 0 or WIDTH <= xpos:
                continue
            if ypos < 0 or HEIGHT <= ypos:
                continue

            if field[ypos][xpos] == EMPTY:
                field[ypos][xpos] = OPENED
                OPEN_COUNT += 1

                count = num_of_bomb(field, xpos, ypos)
                if count == 0 and not (xpos == x_pos and ypos == y_pos):
                    open_tile(field, xpos, ypos)


def main():
    smallfont = pygame.font.SysFont(None, 36)
    largefont = pygame.font.SysFont(None, 72)

    message_clear = largefont.render("!!CLEARED!!", True, (0, 255, 225))
    message_over = largefont.render("GAME OVER!!", True, (0, 255, 225))
    message_restart = smallfont.render("Press R to restart", True, (255, 255, 255))

    message_rect = message_clear.get_rect()
    message_rect.center = (WIDTH * SIZE / 2, HEIGHT * SIZE / 2)

    field = init_game()
    game_over = False

    while True:
        cleared = (OPEN_COUNT == WIDTH * HEIGHT - NUM_OF_BOMBS)

        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()

            if event.type == KEYDOWN:
                if event.key == pygame.K_r:
                    field = init_game()
                    game_over = False

            if event.type == MOUSEBUTTONDOWN and event.button == 1:
                # 게임오버 또는 클리어 이후에는 타일 열기 금지
                if game_over or cleared:
                    continue

                xpos = floor(event.pos[0] / SIZE)
                ypos = floor(event.pos[1] / SIZE)

                if field[ypos][xpos] == BOMB:
                    game_over = True
                else:
                    open_tile(field, xpos, ypos)

        SURFACE.fill((0, 0, 0))

        for ypos in range(HEIGHT):
            for xpos in range(WIDTH):
                tile = field[ypos][xpos]
                rect = (xpos * SIZE, ypos * SIZE, SIZE, SIZE)

                if tile == EMPTY or tile == BOMB:
                    pygame.draw.rect(SURFACE, (192, 192, 192), rect)
                    if game_over and tile == BOMB:
                        pygame.draw.ellipse(SURFACE, (225, 225, 0), rect)

                elif tile == OPENED:
                    pygame.draw.rect(SURFACE, (80, 80, 80), rect)
                    count = num_of_bomb(field, xpos, ypos)
                    if count > 0:
                        num_image = smallfont.render(str(count), True, (255, 255, 0))
                        SURFACE.blit(num_image, (xpos * SIZE + 10, ypos * SIZE + 10))

        for index in range(0, WIDTH * SIZE, SIZE):
            pygame.draw.line(SURFACE, (96, 96, 96), (index, 0), (index, HEIGHT * SIZE))

        for index in range(0, HEIGHT * SIZE, SIZE):
            pygame.draw.line(SURFACE, (96, 96, 96), (0, index), (WIDTH * SIZE, index))

        if cleared:
            SURFACE.blit(message_clear, message_rect.topleft)
            SURFACE.blit(message_restart, (WIDTH * SIZE / 2 - 100, HEIGHT * SIZE / 2 + 50))
        elif game_over:
            SURFACE.blit(message_over, message_rect.topleft)
            SURFACE.blit(message_restart, (WIDTH * SIZE / 2 - 100, HEIGHT * SIZE / 2 + 50))

        pygame.display.update()
        FPSCLOCK.tick(15)


if __name__ == '__main__':
    main()
```