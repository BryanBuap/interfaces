codigo de python para comunicacion de puerto serial
import tkinter as tk
from tkinter import ttk
from tkinter import messagebox
import serial
import serial.tools.list_ports

# Inicializar la conexión serial
def connect():
    global ser
    port = port_combobox.get()
    baudrate = baudrate_combobox.get()
    
    try:
        if not port:
            raise ValueError("El puerto serial no está especificado")
        ser = serial.Serial(port, int(baudrate))
        connection_status_label.config(text="Conectado", foreground="green")
    except Exception as e:
        messagebox.showerror("Error", f"No se pudo conectar: {e}")
        connection_status_label.config(text="Desconectado", foreground="red")

def disconnect():
    global ser
    if ser and ser.is_open:
        ser.close()
    connection_status_label.config(text="Desconectado", foreground="red")

def led1_on():
    if ser and ser.is_open:
        ser.write(b'LED1_ON')  # Cambiado a LED1_ON

def led1_off():
    if ser and ser.is_open:
        ser.write(b'LED1_OFF')  # Cambiado a LED1_OFF

def led2_on():
    if ser and ser.is_open:
        ser.write(b'LED2_ON')  # Comando para LED2

def led2_off():
    if ser and ser.is_open:
        ser.write(b'LED2_OFF')  # Comando para LED2

def send_command():
    if ser and ser.is_open:
        option = option_spinbox.get()
        if option == "1":
            text = input_text.get()
            ser.write(f'OPTION1:{text}'.encode())
            response = ser.readline().decode().strip()  # Leer respuesta del Arduino
            messagebox.showinfo("Respuesta", f"Respuesta del Arduino: {response}")
        elif option == "2":
            ser.write(b'OPTION2')
            response = ser.readline().decode().strip()  # Leer respuesta del Arduino
            messagebox.showinfo("Valor Analógico", f"Valor leído: {response}")
        elif option == "3":
            value = intensity_entry.get()
            try:
                value = int(value)
                if 1 <= value <= 255:
                    ser.write(f'OPTION3:{value}'.encode())
                else:
                    messagebox.showwarning("Advertencia", "El valor debe estar entre 1 y 255")
            except ValueError:1
            messagebox.showwarning("Advertencia", "El valor debe ser un número entero")
        else:
            messagebox.showwarning("Advertencia", "Selecciona una opción válida")

def update_ports():
    ports = [port.device for port in serial.tools.list_ports.comports()]
    port_combobox['values'] = ports
    if ports:
        port_combobox.set(ports[0])  # Selecciona el primer puerto disponible

# Crear la ventana principal
root = tk.Tk()
root.title("Interfaz de Control Arduino")

# Crear el Notebook (pestañas)
notebook = ttk.Notebook(root)
notebook.pack(fill='both', expand=True)

# Pestaña 1: Conectar al puerto
tab1 = ttk.Frame(notebook)
notebook.add(tab1, text='Conexión')

# Elementos de la Pestaña 1
ttk.Label(tab1, text="Puerto:").grid(column=0, row=0, padx=5, pady=5)
port_combobox = ttk.Combobox(tab1)
port_combobox.grid(column=1, row=0, padx=5, pady=5)

ttk.Label(tab1, text="Baudrate:").grid(column=0, row=1, padx=5, pady=5)
baudrate_combobox = ttk.Combobox(tab1, values=["9600", "115200", "38400"])
baudrate_combobox.grid(column=1, row=1, padx=15, pady=5)
baudrate_combobox.set("9600")

connect_button1 = ttk.Button(tab1, text="Conectar", command=connect)
connect_button1.grid(column=0, row=2, padx=15, pady=5)

disconnect_button1 = ttk.Button(tab1, text="Desconectar", command=disconnect)
disconnect_button1.grid(column=1, row=2, padx=15, pady=5)

connection_status_label = ttk.Label(tab1, text="Desconectado", foreground="red")
connection_status_label.grid(column=0, row=3, columnspan=2, pady=5)

# Botones para controlar los LEDs
led_on_button1 = ttk.Button(tab1, text="LED ON 1", command=led1_on)
led_on_button1.grid(column=0, row=4, padx=5, pady=5)

led_off_button1 = ttk.Button(tab1, text="LED OFF 1", command=led1_off)
led_off_button1.grid(column=1, row=4, padx=5, pady=5)

led_on_button2 = ttk.Button(tab1, text="LED ON 2", command=led2_on)  # Botón para LED2
led_on_button2.grid(column=0, row=5, padx=5, pady=5)

led_off_button2 = ttk.Button(tab1, text="LED OFF 2", command=led2_off)  # Botón para LED2
led_off_button2.grid(column=1, row=5, padx=5, pady=5)

# Botón para actualizar la lista de puertos
update_ports_button = ttk.Button(tab1, text="Actualizar Puertos", command=update_ports)
update_ports_button.grid(column=2, row=5, padx=5, pady=5)

# Pestaña 2: Controlar tareas
tab2 = ttk.Frame(notebook)
notebook.add(tab2, text='Control de Tareas')

# Elementos de la Pestaña 2
ttk.Label(tab2, text="Seleccionar opción:").grid(column=0, row=0, padx=5, pady=5)
option_spinbox = ttk.Spinbox(tab2, from_=1, to=4)
option_spinbox.grid(column=1, row=0, padx=5, pady=5)

input_text = tk.Entry(tab2)
input_text.grid(column=0, row=1, columnspan=2, padx=5, pady=5)

intensity_entry = tk.Entry(tab2)
intensity_entry.grid(column=0, row=2, columnspan=2, padx=5, pady=5) 

send_button = ttk.Button(tab2, text="Enviar", command=send_command)
send_button.grid(column=0, row=3, columnspan=2, pady=5)

root.mainloop()
