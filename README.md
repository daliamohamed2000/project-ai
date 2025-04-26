import pygame
pygame.init()
WIDTH, HEIGHT =800 , 800
SQUARE_SIZE = WIDTH // 8

WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
LIGHT_BROWN = (240, 217, 181)
DARK_BROWN = (120, 72, 36)
font = pygame.font.Font(None, 36)


black_queen = pygame.image.load('C:/Users/mr/Downloads/black queen.png'); black_queen = pygame.transform.scale(black_queen, (80, 80))
black_king = pygame.image.load('C:/Users/mr/Downloads/black king.png'); black_king = pygame.transform.scale(black_king, (80, 80))
black_rook = pygame.image.load('C:/Users/mr/Downloads/black rook.png'); black_rook = pygame.transform.scale(black_rook, (80, 80))
black_bishop = pygame.image.load('C:/Users/mr/Downloads/black bishop.png'); black_bishop = pygame.transform.scale(black_bishop, (80, 80))
black_knight = pygame.image.load('C:/Users/mr/Downloads/black knight.png'); black_knight = pygame.transform.scale(black_knight, (80, 80))
black_pawn = pygame.image.load('C:/Users/mr/Downloads/black pawn.png'); black_pawn = pygame.transform.scale(black_pawn, (65, 65))
white_queen = pygame.image.load('C:/Users/mr/Downloads/white queen.png'); white_queen = pygame.transform.scale(white_queen, (80, 80))
white_king = pygame.image.load('C:/Users/mr/Downloads/white king.png'); white_king = pygame.transform.scale(white_king, (80, 80))
white_rook = pygame.image.load('C:/Users/mr/Downloads/white rook.png'); white_rook = pygame.transform.scale(white_rook, (80, 80))
white_bishop = pygame.image.load('C:/Users/mr/Downloads/white bishop.png'); white_bishop = pygame.transform.scale(white_bishop, (80, 80))
white_knight = pygame.image.load('C:/Users/mr/Downloads/white knight.png'); white_knight = pygame.transform.scale(white_knight, (80, 80))
white_pawn = pygame.image.load('C:/Users/mr/Downloads/white pawn.png'); white_pawn = pygame.transform.scale(white_pawn, (65, 65))

PIECE_IMAGES = {
    'BR': black_rook, 'BN': black_knight, 'BB': black_bishop, 'BQ': black_queen, 'BK': black_king, 'BP': black_pawn,
    'WR': white_rook, 'WN': white_knight, 'WB': white_bishop, 'WQ': white_queen, 'WK': white_king, 'WP': white_pawn
}

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Chess Game")

def initialize_board():
    return [
        ['BR', 'BN', 'BB', 'BQ', 'BK', 'BB', 'BN', 'BR'],
        ['BP', 'BP', 'BP', 'BP', 'BP', 'BP', 'BP', 'BP'],
        ['--', '--', '--', '--', '--', '--', '--', '--'],
        ['--', '--', '--', '--', '--', '--', '--', '--'],
        ['--', '--', '--', '--', '--', '--', '--', '--'],
        ['--', '--', '--', '--', '--', '--', '--', '--'],
        ['WP', 'WP', 'WP', 'WP', 'WP', 'WP', 'WP', 'WP'],
        ['WR', 'WN', 'WB', 'WQ', 'WK', 'WB', 'WN', 'WR'],
    ]

def draw_board():
    for row in range(8):
        for col in range(8):
            color = LIGHT_BROWN if (row + col) % 2 == 0 else DARK_BROWN
            pygame.draw.rect(screen, color, (col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE))

def draw_pieces(board):
    for row in range(8):
        for col in range(8):
            piece = board[row][col]
            if piece != '--':
                screen.blit(PIECE_IMAGES[piece], (col * SQUARE_SIZE + 10, row * SQUARE_SIZE + 10))

def get_square_under_mouse():
    x, y = pygame.mouse.get_pos()
    row = y // SQUARE_SIZE
    col = x // SQUARE_SIZE
    return row, col

def validate_straight_move(board, start, end):
    start_row, start_col = start
    end_row, end_col = end
    if start_row == end_row:
        step = 1 if start_col < end_col else -1
        for col in range(start_col + step, end_col, step):
            if board[start_row][col] != '--':
                return False
        return True
    if start_col == end_col:
        step = 1 if start_row < end_row else -1
        for row in range(start_row + step, end_row, step):
            if board[row][start_col] != '--':
                return False
        return True
    return False

def validate_diagonal_move(board, start, end):
    start_row, start_col = start
    end_row, end_col = end
    if abs(start_row - end_row) != abs(start_col - end_col):
        return False
    row_step = 1 if start_row < end_row else -1
    col_step = 1 if start_col < end_col else -1
    for step in range(1, abs(start_row - end_row)):
        if board[start_row + step * row_step][start_col + step * col_step] != '--':#####
            return False
    return True

def promote_pawn(screen,piece_color, board, start, end):
    """Handle pawn promotion within Pygame."""
    # Create a surface for the promotion menu
    menu_width, menu_height = 200, 200
    menu_x = (WIDTH - menu_width) // 2
    menu_y = (HEIGHT - menu_height) // 2
    menu_rect = pygame.Rect(menu_x, menu_y, menu_width, menu_height)

    # Draw the promotion menu
    pygame.draw.rect(screen, WHITE, menu_rect)
    pygame.draw.rect(screen, BLACK, menu_rect, 2)  # Border

    # Define promotion options
    options = ['Q-Queen', 'R-Rook', 'B-Bishop', 'N-knight']
    option_buttons = []
    button_width, button_height = 80, 40
    spacing = 10

    for i, option in enumerate(options):
        button_x = menu_x + (menu_width - button_width) // 2
        button_y = menu_y + spacing + i * (button_height + spacing)
        button_rect = pygame.Rect(button_x, button_y, button_width, button_height)
        option_buttons.append((button_rect, option))
        pygame.draw.rect(screen, DARK_BROWN, button_rect)
        pygame.draw.rect(screen, BLACK, button_rect, 2)  # Border
        text = font.render(option, True, WHITE)
        text_rect = text.get_rect(center=button_rect.center)
        screen.blit(text, text_rect)

    pygame.display.flip()

    # Wait for the player to select an option
    while True:
        for event in pygame.event.get():
            if event.type == pygame.MOUSEBUTTONDOWN:
                mouse_pos = pygame.mouse.get_pos()
                for button_rect, option in option_buttons:
                    if button_rect.collidepoint(mouse_pos):
                        # Update the board with the selected option
                        board[end[0]][end[1]] = piece_color + option[0]
                        board[start[0]][start[1]] = '--'
                        return


def is_valid_move(board, start, end, current_turn):
    """Check if a move is valid."""
    start_row, start_col = start
    end_row, end_col = end
    piece = board[start_row][start_col]

    if piece == '--':
        return False  # No piece to move

    # Ensure the piece belongs to the current player
    if current_turn == 'white' and piece[0] != 'W':
        return False
    if current_turn == 'black' and piece[0] != 'B':
        return False

    # Prevent capturing same-color pieces
    target_piece = board[end_row][end_col]
    #eat my self
    if target_piece != '--' and target_piece[0] == piece[0]:
        return False

    # Calculate movement deltas
    row_delta = end_row - start_row#*****************
    col_delta = end_col - start_col

    # Pawn movement rules
    if piece[1] == 'P':
        direction = -1 if piece[0] == 'W' else 1
        start_row, start_col = start
        end_row, end_col = end

        # Forward move
        if start_col == end_col and board[end_row][end_col] == '--':
            if end_row == start_row + direction:
                return True
            if end_row == start_row + 2 * direction and (start_row == 1 or start_row == 6):
                return board[start_row + direction][start_col] == '--'
        # eating move
        if abs(end_col - start_col) == 1 and end_row == start_row + direction:
            return board[end_row][end_col] != '--' and board[end_row][end_col][0] != piece[0]
        return False

    # Knight movement rules horse
    if piece[1] == 'N':
        if (abs(row_delta), abs(col_delta)) in [(2, 1), (1, 2)]:
            return True
        return False

    # Rook movement rules
    if piece[1] == 'R':
        if row_delta == 0 or col_delta == 0:
            if validate_straight_move(board, start, end):
                return True
        return False

    # Bishop movement rules
    if piece[1] == 'B':
        if abs(row_delta) == abs(col_delta):
            if validate_diagonal_move(board, start, end):
                return True
        return False

    # Queen movement rules
    if piece[1] == 'Q':
        if row_delta == 0 or col_delta == 0 or abs(row_delta) == abs(col_delta):
            return validate_straight_move(board, start, end) or validate_diagonal_move(board, start, end)

    # King movement rules
    if piece[1] == 'K':
        row_diff, col_diff = abs(start[0] - end[0]), abs(start[1] - end[1])
        return max(row_diff, col_diff) == 1

    return False


def is_check(board, current_player):
    """
    Checks if the current player's king is in check.
    """
    king_pos = None
    enemy_pieces = []

    for row in range(8):
        for col in range(8):
            piece = board[row][col]
            if piece == ('WK' if current_player == "white" else 'BK'):
                king_pos = (row, col)
            else:
                piece='B' if current_player == "white" else 'W'
                enemy_pieces.append((piece, (row, col)))

    for piece, pos in enemy_pieces:
        if is_valid_move(board, pos, king_pos, piece[0]):
            return True
    return False


def has_legal_moves(board, current_player):

    """
    Checks if the current player has any legal moves.
    """
    for row in range(8):
        for col in range(8):
            piece = board[row][col]
            if piece.startswith('W' if current_player == "white" else 'B'):
                for r in range(8):
                    for c in range(8):
                        start = (row, col)
                        end = (r, c)
                        if is_valid_move(board, start, end, current_player):
                            # Simulate the move
                            original_piece = board[end[0]][end[1]]
                            #
                            piece = board[start[0]][start[1]]
                            board[end[0]][end[1]] = piece
                            board[start[0]][start[1]] = '--'
                            #
                            is_still_check = is_check(board, current_player)
                            # Undo the move
                            board[end[0]][end[1]] = original_piece
                            board[start[0]][start[1]] = piece
                            if not is_still_check:
                                return True
    return False

def is_king_captured(board, player_color):
    """Check if the given player's king is captured."""
    king_symbol = 'WK' if player_color == 'white' else 'BK'
    for row in range(8):
        for col in range(8):
            if board[row][col] == king_symbol:
                return False  # King is still on the board
    return True  # King is captured


def is_draw(board):
        pieces = [cell for row in board for cell in row if cell not in ('--', None)]
        # If only kings are left
        if len(pieces) == 2:
            return True
        # If only a king and a bishop or knight remain
        if len(pieces) == 3 and any(piece in ['WN', 'BN', 'WB', 'BB'] for piece in pieces):
            return True
        return False

import random

def display_winner(winner, message, sound):
    # تهيئة pygame
    pygame.init()
    screen = pygame.display.set_mode((800, 800))
    pygame.display.set_caption("Winner Screen")

    # إعداد الخط
    font = pygame.font.Font(None, 74)
    text = font.render(message, True, (255, 215, 0))
    text_rect = text.get_rect(center=(800 // 2, 800 // 2))

    # تحضير النجوم
    stars = [
        [random.randint(0, 800), random.randint(0, 800), random.randint(2, 5)]
        for _ in range(100)
    ]

    # تشغيل الصوت باستخدام pygame.mixer
    pygame.mixer.init()
    pygame.mixer.music.load(sound)
    pygame.mixer.music.play()

    # رسم النجوم مع النص
    clock = pygame.time.Clock()
    running = True
    while running:
        screen.fill((0, 0, 0))  # مسح الشاشة باللون الأسود

        for star in stars:
            pygame.draw.circle(screen, (255, 255, 255), (star[0], star[1]), star[2])
            star[1] += 2
            if star[1] > 800:
                star[1] = 0
                star[0] = random.randint(0, 800)

        screen.blit(text, text_rect)  # عرض النص
        pygame.display.flip()  # تحديث الشاشة

        clock.tick(30)  # تحديد معدل الإطارات (30 إطار بالثانية)

        # إنهاء الحلقة عندما يتوقف الصوت
        if not pygame.mixer.music.get_busy():
            running = False

    pygame.quit()


king_has_moved = {'W': False, 'B': False}
rook_has_moved = {'W_kingside': False, 'B_kingside': False, 'W_queenside': False, 'B_queenside': False}


def castling_move(board, start, end, current_player, king_has_moved, rook_has_moved):
    piece = board[start[0]][start[1]]

    if piece[1] != 'K':  # Ensure it's the king
        return False

    color = piece[0]
    if (color == 'W' and current_player != "white") or (color == 'B' and current_player != "black"):
        return False  # Ensure it's the correct player's turn

    # Kingside castling (e1 to g1 or e8 to g8)
    if start[1] == 4 and end[1] == 6:
        if king_has_moved[color] or rook_has_moved[color + '_kingside']:
            return False  # Ensure the king and rook have not moved
        if board[start[0]][7] == color + 'R' and board[start[0]][5] == '--' and board[start[0]][6] == '--':
            if not is_check(board, current_player):
                board[end[0]][end[1]] = piece
                board[start[0]][start[1]] = '--'
                board[start[0]][7] = '--'
                board[start[0]][5] = color + 'R'
                king_has_moved[color] = True
                rook_has_moved[color + '_kingside'] = True
                return True

    # Queenside castling (e1 to c1 or e8 to c8)
    elif start[1] == 4 and end[1] == 2:
        if king_has_moved[color] or rook_has_moved[color + '_queenside']:
            return False  # Ensure the king and rook have not moved
        if board[start[0]][0] == color + 'R' and board[start[0]][1] == '--' and board[start[0]][2] == '--' and \
                board[start[0]][3] == '--':
            if not is_check(board, current_player):
                board[end[0]][end[1]] = piece
                board[start[0]][start[1]] = '--'
                board[start[0]][0] = '--'
                board[start[0]][3] = color + 'R'
                king_has_moved[color] = True
                rook_has_moved[color + '_queenside'] = True
                return True

    return False

def main():
    board = initialize_board()
    current_turn = 'white'  # Start with white's turn
    selected_piece = None
    running = True

    while running:
        #while interact with board
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN:
                row, col = get_square_under_mouse()
                if selected_piece:
                    # Try to move the selected piece
                    target_row, target_col = row, col
                    if is_valid_move(board, selected_piece, (target_row, target_col), current_turn):
                        board[target_row][target_col] = board[selected_piece[0]][selected_piece[1]]
                        board[selected_piece[0]][selected_piece[1]] = '--'

                        if board[target_row][target_col][1] == 'K':  # If it's a king
                            king_has_moved[board[target_row][target_col][0]] = True

                            # Track rook movement
                        if board[target_row][target_col][1] == 'R':  # If it's a rook
                            if target_row == 0:  # Leftmost rook (a-file)
                                rook_has_moved[board[target_row][target_col][0] + '_queenside'] = True
                            elif target_row == 7:  # Rightmost rook (h-file)
                                rook_has_moved[board[target_row][target_col][0] + '_kingside'] = True

                        if board[target_row][target_col][1] == 'P':
                            if (current_turn[0] == 'w' and target_row == 0) or (current_turn[0] == 'b' and target_row == 7):
                                promote_pawn(screen, current_turn[0].upper() , board, selected_piece, (target_row, target_col))
                        # Check if the opponent's king is captured


                        if is_king_captured(board, 'white' if current_turn == 'black' else 'black'):
                            message=f"{current_turn.upper()} WINS!"
                            sound='C:/Users/mr/Downloads/level-win-6416.mp3'
                            display_winner(current_turn, message, sound)
                            exit()
                        elif is_draw(board):
                            message = " IT IS A DRAW!"
                            sound='C:/Users/mr/Downloads/bell-172780.mp3'
                            display_winner(current_turn, message, sound)
                            exit()


                        current_turn = 'black' if current_turn == 'white' else 'white'  # Switch turns
                    elif castling_move(board, selected_piece, (target_row, target_col), current_turn, king_has_moved, rook_has_moved):
                        current_turn = 'black' if current_turn == 'white' else 'white'  # Switch turns

                    selected_piece = None  # Deselect after attempting a move

                    if is_check(board, current_turn):
                        if not has_legal_moves(board, current_turn):
                            current_turn = 'white' if current_turn == 'black' else 'black'
                            message = f"{current_turn.upper()} WINS!"
                            sound = 'C:/Users/mr/Downloads/level-win-6416.mp3'
                            display_winner(current_turn, message, sound)
                            exit()
                        else:
                            pygame.mixer.Sound('C:/Users/mr/Downloads/error_sound-221445.mp3').play()
                    elif not has_legal_moves(board, current_turn):
                        message = " IT IS A DRAW!"
                        sound = 'C:/Users/mr/Downloads/bell-172780.mp3'
                        display_winner(current_turn, message, sound)
                        exit()
                else:
                    # Select a piece
                    if board[row][col] != '--' and board[row][col][0].upper() == current_turn[0].upper():
                        selected_piece = (row, col)

        draw_board()
        draw_pieces(board)

        # Highlight selected square
        if selected_piece:
            #green light
            pygame.draw.rect(
                screen, (0, 255, 0),
                (selected_piece[1] * SQUARE_SIZE, selected_piece[0] * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE), 3
            )

        pygame.display.flip()

    pygame.quit()

if __name__ == "__main__":
    main()
