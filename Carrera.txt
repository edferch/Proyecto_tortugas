import turtle
import random
from datetime import datetime
import time

SCREEN_WIDTH = 600
SCREEN_HEIGHT = 400
X_START = -250
X_FINISH = 250
Y_POSITIONS = [125, 75, 25, -25, -75, -125]
COLORS = ["red", "orange", "yellow", "green", "blue", "purple", "pink", "brown"]
FILE_NAME = "resultados.txt"

_main_screen = None

def setup_screen():
    global _main_screen
    
    if _main_screen is None:
        _main_screen = turtle.Screen()
        _main_screen.setup(SCREEN_WIDTH, SCREEN_HEIGHT)
    else:
        _main_screen.reset() 
    
    _main_screen.title("Carrera de Tortugas")
    _main_screen.bgcolor("#88FF00")
    return _main_screen

def draw_track():
    pen = turtle.Turtle()
    pen.speed(0)
    pen.hideturtle()
    pen.penup()
    
    pen.color("black")
    pen.goto(X_FINISH, 150)
    pen.write(" META", align="left", font=("Arial", 10, "bold"))
    pen.pendown()
    pen.goto(X_FINISH, -150)
    
    pen.penup()
    pen.goto(X_START, 150)
    pen.write("INICIO ", align="right", font=("Arial", 10, "bold"))
    pen.pendown()
    pen.goto(X_START, -150)

    pen.color("white")
    lane_y_positions = [100, 50, 0, -50, -100]

    for y in lane_y_positions:
        pen.penup()
        pen.goto(X_START, y)
        
        track_length = X_FINISH - X_START
        segment_length = 20
        num_segments = track_length // segment_length

        for _ in range(num_segments):
            pen.pendown()
            pen.forward(10)
            pen.penup()
            pen.forward(10)

def get_race_config(screen):
    num_turtles = 0
    while True:
        try:
            num_turtles = int(screen.numinput(
                "Tortugas", 
                "Número de tortugas (2-6)", 
                minval=2, 
                maxval=6
            ))
            break
        except TypeError:
            return None, None

    selected_colors = []
    available_colors = COLORS[:]
    
    for i in range(num_turtles):
        color_prompt = f"Color para tortuga {i+1}:\n{', '.join(available_colors)}"
        color = ""
        while True:
            color = screen.textinput("Color", color_prompt)
            if color is None:
                return None, None
            
            color = color.lower().strip()
            
            if color in available_colors:
                selected_colors.append(color)
                available_colors.remove(color)
                break
            else:
                color_prompt = f"Color no valido\n{', '.join(available_colors)}"
                
    return num_turtles, selected_colors

def create_turtles(colors):
    turtles = []
    for i, color in enumerate(colors):
        t = turtle.Turtle(shape="turtle")
        t.shapesize(1.5)
        t.color(color)
        t.penup()
        t.goto(X_START, Y_POSITIONS[i])
        t.pendown()
        turtles.append(t)
    return turtles

def start_race(turtles, colors):
    winner = None
    while winner is None:
        for i, t in enumerate(turtles):
            distance = random.randint(1, 15)
            t.forward(distance)
            
            if t.xcor() > X_FINISH:
                winner = colors[i]
                break
        
        if winner:
            break
            
    return winner

def show_winner(winner_color):
    pen = turtle.Turtle()
    pen.hideturtle()
    pen.penup()
    pen.goto(0, 0)
    pen.color(winner_color)
    pen.write(
        f"La tortuga {winner_color.upper()} ha ganado", 
        align="center", 
        font=("Arial", 22, "bold")
    )
    
    time.sleep(4)

def save_results(winner, colors):
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    participants = ", ".join([c.capitalize() for c in colors])
    
    try:
        with open(FILE_NAME, "a", encoding="utf-8") as f:
            f.write("="*40 + "\n")
            f.write(f"Fecha y hora: {now}\n")
            f.write(f"Participantes: {participants}\n")
            f.write(f"Ganador: {winner.capitalize()}\n")
            f.write("="*40 + "\n\n")
        print(f"\nResultados guardados")
    except IOError as e:
        print(f"Error al guardar")

def run_new_race():
    screen = setup_screen()
    draw_track()
    
    num_turtles, selected_colors = get_race_config(screen)
    
    if num_turtles is None:
        print("Carrera cancelada")
        return

    print(f"Carrea iniciada con {num_turtles} tortugas: {', '.join(selected_colors)}")
    
    all_turtles = create_turtles(selected_colors)
    
    time.sleep(1) 
    
    winner = start_race(all_turtles, selected_colors)
    
    print(f"{winner} ha ganado")
    
    save_results(winner, selected_colors)
    show_winner(winner)
    
def show_history():
    print("\n" + "==HISTORIAL DE CARRERAS==".center(40))
    try:
        with open(FILE_NAME, "r", encoding="utf-8") as f:
            history = f.read()
            if not history:
                print("No hay carreras guardadas")
            else:
                print(history)
    except FileNotFoundError:
        print(f"No se encontro el archivo")
    except IOError as e:
        print(f"Error al leer {e}")
    print("-" * 40)
    input("Presiona Enter para volver al menu")

def main():
    while True:
        print("\n" + "="*30)
        print("=====CARRERA DE TORTUGAS=====")
        print("="*30)
        print("1. Iniciar carrera")
        print("2. Historial de carreras")
        print("3. Salir")
        print("="*30)
        
        choice = input("Selecciona una opcion : ").strip()
        
        if choice == '1':
            run_new_race()
        elif choice == '2':
            show_history()
        elif choice == '3':
            print("Saliendo")
            try:
                turtle.bye()
            except turtle.Terminator:
                pass
            break
        else:
            print("Opcion no valida")

if __name__ == "__main__":
    main()