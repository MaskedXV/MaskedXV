import pygame
import random

# Define constants
SCREEN_WIDTH = 300
SCREEN_HEIGHT = 600
BLOCK_SIZE = 30
GRID_WIDTH = SCREEN_WIDTH // BLOCK_SIZE
GRID_HEIGHT = SCREEN_HEIGHT // BLOCK_SIZE
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
SHAPE_COLORS = [(255, 0, 0), (0, 255, 0), (0, 0, 255), (255, 255, 0), (255, 0, 255), (0, 255, 255), (128, 0, 0)]

SHAPES = [
    [[1, 1, 1],
     [0, 1, 0]],

    [[1, 1, 0],
     [0, 1, 1]],

    [[1, 1],
     [1, 1]],

    [[1, 1, 1, 1]],

    [[1, 1, 1],
     [1, 0, 0]],

    [[1, 1, 1],
     [0, 0, 1]],

    [[0, 1, 1],
     [1, 1, 0]]
]

class Tetris:
    def __init__(self):
        self.grid = [[0] * GRID_WIDTH for _ in range(GRID_HEIGHT)]
        self.current_piece = self.new_piece()
        self.score = 0

    def new_piece(self):
        shape = random.choice(SHAPES)
        piece = {'shape': shape,
                 'x': GRID_WIDTH // 2 - len(shape[0]) // 2,
                 'y': 0,
                 'color': random.choice(SHAPE_COLORS)}
        return piece

    def draw_grid(self, surface):
        for y, row in enumerate(self.grid):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(surface, cell, (x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE))

    def draw_piece(self, surface):
        shape = self.current_piece['shape']
        x, y = self.current_piece['x'], self.current_piece['y']
        color = self.current_piece['color']
        for row in range(len(shape)):
            for col in range(len(shape[0])):
                if shape[row][col]:
                    pygame.draw.rect(surface, color, ((x + col) * BLOCK_SIZE, (y + row) * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE))

    def valid_position(self, shape, x, y):
        for row in range(len(shape)):
            for col in range(len(shape[0])):
                if shape[row][col]:
                    if x + col < 0 or x + col >= GRID_WIDTH or y + row >= GRID_HEIGHT:
                        return False
                    if self.grid[y + row][x + col]:
                        return False
        return True

    def freeze_piece(self):
        shape = self.current_piece['shape']
        x, y = self.current_piece['x'], self.current_piece['y']
        color = self.current_piece['color']
        for row in range(len(shape)):
            for col in range(len(shape[0])):
                if shape[row][col]:
                    self.grid[y + row][x + col] = color

    def check_lines(self):
        lines_to_remove = [i for i, row in enumerate(self.grid) if all(row)]
        for row in lines_to_remove:
            del self.grid[row]
            self.grid.insert(0, [0] * GRID_WIDTH)
        self.score += len(lines_to_remove) ** 2

    def collide(self):
        shape = self.current_piece['shape']
        x, y = self.current_piece['x'], self.current_piece['y']
        for row in range(len(shape)):
            for col in range(len(shape[0])):
                if shape[row][col]:
                    if y + row >= GRID_HEIGHT - 1 or self.grid[y + row + 1][x + col]:
                        return True
        return False

    def rotate_piece(self):
        shape = self.current_piece['shape']
        rotated_shape = [[shape[col][row] for col in range(len(shape))] for row in range(len(shape[0]) - 1, -1, -1)]
        if self.valid_position(rotated_shape, self.current_piece['x'], self.current_piece['y']):
            self.current_piece['shape'] = rotated_shape

    def move_piece(self, dx):
        if self.valid_position(self.current_piece['shape'], self.current_piece['x'] + dx, self.current_piece['y']):
            self.current_piece['x'] += dx

    def drop_piece(self):
        while not self.collide():
            self.current_piece['y'] += 1
        self.current_piece['y'] -= 1

    def game_over(self):
        return any(self.grid[0])

def main():
    pygame.init()
    screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
    pygame.display.set_caption('Tetris')
    clock = pygame.time.Clock()
    tetris = Tetris()
    game_over = False
    move_down = False
    while not game_over:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_over = True
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT:
                    tetris.move_piece(-1)
                elif event.key == pygame.K_RIGHT:
                    tetris.move_piece(1)
                elif event.key == pygame.K_DOWN:
                    move_down = True
                elif event.key == pygame.K_UP:
                    tetris.rotate_piece()
                elif event.key == pygame.K_SPACE:
                    tetris.drop_piece()
        screen.fill(BLACK)
        if move_down:
            tetris.drop_piece()
        if tetris.collide():
            tetris.freeze_piece()
            tetris.check_lines()
            if tetris.game_over():
                game_over = True
            else:
                tetris.current_piece = tetris.new_piece()
                move_down = False
        tetris.draw_grid(screen)
        tetris.draw_piece(screen)
        pygame.display.update()
        clock.tick(10)

    pygame.quit()

if __name__ == '__main__':
    main()
