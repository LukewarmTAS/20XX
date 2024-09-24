import argparse
import signal
import sys
import melee

def check_port(value):
    ivalue = int(value)
    if ivalue < 1 or ivalue > 4:
        raise argparse.ArgumentTypeError("%s is an invalid controller port. \
                                         Must be 1, 2, 3, or 4." % value)
    return ivalue

parser = argparse.ArgumentParser(description='Example of libmelee in action')
parser.add_argument('--port', '-p', type=check_port,
                    help='The controller port (1-4) your AI will play on',
                    default=2)
parser.add_argument('--debug', '-d', action='store_true',
                    help='Debug mode. Creates a CSV of all game states')
parser.add_argument('--address', '-a', default="127.0.0.1",
                    help='IP address of Slippi/Wii')
parser.add_argument('--dolphin_executable_path', '-e', default="C:\\Users\\lbonc\\AppData\\Roaming\\Slippi Launcher\\netplay\\",
                    help='The directory where dolphin is')
parser.add_argument('--connect_code', '-t', default="",

                    help='Direct connect code to connect to in Slippi Online')
parser.add_argument('--iso', default="D:\\Melee\\Super Smash Bros. Melee (USA) (En,Ja) (Rev 2).ciso", type=str,
                    help='Path to melee iso.')

args = parser.parse_args()

log = None
if args.debug:
    log = melee.Logger()

console = melee.Console(path=args.dolphin_executable_path,
                        slippi_address=args.address,
                        logger=log)

controller = melee.Controller(console=console, port=args.port, type=melee.ControllerType.STANDARD)

def signal_handler(sig, frame):
    console.stop()
    if args.debug:
        log.writelog()
        print("")
        print("Log file created: " + log.filename)
    print("Shutting down cleanly...")
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)

console.run(iso_path=args.iso)

print("Connecting to console...")
if not console.connect():
    print("ERROR: Failed to connect to the console.")
    sys.exit(-1)
print("Console connected")

print("Connecting controller to console...")
if not controller.connect():
    print("ERROR: Failed to connect the controller.")
    sys.exit(-1)

print("Controller connected")

framedata = melee.framedata.FrameData()
costume = 0

wavedash_timestamp = 0
shine_cancel_timestamp = 0

def calculate_inputs(gamestate, bot_port, opponent_port):
    global wavedash_timestamp
    global shine_cancel_timestamp
    inputs = [False, False, False, False, False, 0.5, 0.5, 0.5, 0.5]
    if gamestate.frame - wavedash_timestamp == 1:
        inputs[4] = True
        inputs[6] = 0.0
        if gamestate.players[bot_port].position.x > gamestate.players[opponent_port].position.x:
            inputs[5] = 0.0
        elif gamestate.players[bot_port].position.x < gamestate.players[opponent_port].position.x:
            inputs[5] = 1.0
    elif gamestate.frame - shine_cancel_timestamp == 4:
        inputs[2] = True
        wavedash_timestamp = gamestate.frame
    elif gamestate.players[bot_port].position.x - gamestate.players[opponent_port].position.x < 5 and gamestate.players[bot_port].position.x > gamestate.players[opponent_port].position.x and gamestate.players[bot_port].position.x > 0:
        inputs[6] = 0.0
        inputs[1] = True
        shine_cancel_timestamp = gamestate.frame
    elif gamestate.players[opponent_port].position.x - gamestate.players[bot_port].position.x < 5 and gamestate.players[bot_port].position.x < gamestate.players[opponent_port].position.x and gamestate.players[bot_port].position.x < 0:
        inputs[6] = 0.0
        inputs[1] = True
        shine_cancel_timestamp = gamestate.frame
    elif gamestate.players[bot_port].position.x > gamestate.players[opponent_port].position.x:
        inputs[5] = 0.0
    elif gamestate.players[bot_port].position.x < gamestate.players[opponent_port].position.x:
        inputs[5] = 1.0
    return inputs

while True:
    opponent = None
    gamestate = console.step()
    if gamestate is None:
        continue
    if console.processingtime * 1000 > 12:
        print("WARNING: Last frame took " + str(console.processingtime*1000) + "ms to process.")
    if gamestate.menu_state in [melee.Menu.IN_GAME]:
        bot_port = melee.gamestate.port_detector(gamestate, melee.Character.FOX, costume)
        opponent_port = None
        for port in range(4):
            try:
                opponent = gamestate.players[port + 1]
            except: KeyError
            if opponent is None:
                continue
            elif opponent is not gamestate.players[bot_port]:
                opponent_port = port + 1
                break
        if bot_port > 0:
            inputs = calculate_inputs(gamestate, bot_port, opponent_port)
            controller.tilt_analog(melee.enums.Button.BUTTON_MAIN, inputs[5], inputs[6])
            controller.tilt_analog(melee.enums.Button.BUTTON_C, inputs[7], inputs[8])
            if inputs[0] is True:
                controller.press_button(melee.enums.Button.BUTTON_A)
            else:
                controller.release_button(melee.enums.Button.BUTTON_A)
            if inputs[1] is True:
                controller.press_button(melee.enums.Button.BUTTON_B)
            else:
                controller.release_button(melee.enums.Button.BUTTON_B)
            if inputs[2] is True:
                controller.press_button(melee.enums.Button.BUTTON_X)
            else:
                controller.release_button(melee.enums.Button.BUTTON_X)
            if inputs[3] is True:
                controller.press_button(melee.enums.Button.BUTTON_Z)
            else:
                controller.release_button(melee.enums.Button.BUTTON_Z)
            if inputs[4] is True:
                controller.press_button(melee.enums.Button.BUTTON_R)
            else:
                controller.release_button(melee.enums.Button.BUTTON_R)
        if log:
            log.logframe(gamestate)
            log.writeframe()
    else:
        melee.MenuHelper.menu_helper_simple(gamestate, controller, melee.Character.FOX, melee.Stage.FINAL_DESTINATION, args.connect_code, costume=costume, autostart=False, swag=False)
        if log:
            log.skipframe()
