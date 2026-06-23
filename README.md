# Codigo-finanzas-personales
este codigo genera un extracto para manejar finanzas personales

# ─── CAPA 3: BASE DE DATOS (simulada con diccionarios) ───────
usuarios = {
    "juan": {
        "email": "juan@email.com",
        "clave": "1234"
    }
}

movimientos = []  # Lista de movimientos registrados

metas = {
    "juan": {
        "meta_ahorro": 500,
        "acumulado": 40
    }
}

# ─── CAPA 2: LÓGICA DEL SISTEMA ──────────────────────────────

def calcular_distribucion(monto):
    """
    Distribuye automáticamente el monto ingresado:
    40% ahorro, 50% inversión, 10% reserva
    """
    ahorro    = monto * 0.40
    inversion = monto * 0.50
    reserva   = monto * 0.10
    return ahorro, inversion, reserva


def verificar_meta(usuario, nuevo_ahorro):
    """
    Verifica si el usuario alcanzó su meta de ahorro.
    Usa un CONDICIONAL para comparar el acumulado con la meta.
    """
    metas[usuario]["acumulado"] += nuevo_ahorro
    acumulado = metas[usuario]["acumulado"]
    meta      = metas[usuario]["meta_ahorro"]
    porcentaje = (acumulado / meta) * 100

    # ── CONDICIONAL: verificar si alcanzó la meta ──
    if acumulado >= meta:
        return f"🎉 ¡Felicidades! Alcanzaste tu meta de ${meta:.2f}"
    else:
        return f"📊 Meta al {porcentaje:.1f}% — Te faltan ${meta - acumulado:.2f}"


def generar_alerta(monto, limite=200):
    """
    Genera una alerta si el gasto supera el límite permitido.
    Usa un CONDICIONAL para detectar gastos excesivos.
    """
    if monto > limite:
        return f"⚠️  ALERTA: Gasto de ${monto:.2f} supera el límite de ${limite:.2f}"
    else:
        return None


def registrar_movimiento(usuario, monto, tipo):
    """
    Registra un movimiento en la base de datos simulada.
    """
    from datetime import date
    movimiento = {
        "usuario": usuario,
        "monto":   monto,
        "fecha":   str(date.today()),
        "tipo":    tipo   # "ingreso" o "gasto"
    }
    movimientos.append(movimiento)
    return movimiento


# ─── CAPA 4: NOTIFICACIONES ───────────────────────────────────

def enviar_notificaciones(usuario, ahorro, inversion, reserva, estado_meta, alerta):
    """
    Simula el envío de notificaciones al usuario.
    """
    print("\n" + "="*50)
    print("       CAPA 4 — NOTIFICACIONES")
    print("="*50)

    # Notificación por email
    print(f"\n📧 Email enviado a {usuarios[usuario]['email']}:")
    print(f"   'Guardaste ${ahorro:.2f} hoy'")

    # Alerta en pantalla
    print(f"\n🖥️  Alerta en pantalla:")
    print(f"   '{estado_meta}'")

    # Reporte PDF simulado
    print(f"\n📄 Reporte generado:")
    print(f"   Ahorro:    ${ahorro:.2f}")
    print(f"   Inversión: ${inversion:.2f}")
    print(f"   Reserva:   ${reserva:.2f}")

    # Alerta de gasto excesivo si existe
    if alerta:
        print(f"\n{alerta}")


# ─── CAPA 1: INTERFAZ DE USUARIO ─────────────────────────────

def iniciar_sesion():
    """
    Solicita usuario y clave. Usa un BUCLE para permitir
    hasta 3 intentos antes de bloquear el acceso.
    """
    print("\n" + "="*50)
    print("    SISTEMA DE GESTIÓN FINANCIERA PERSONAL")
    print("="*50)
    print("\n🔐 INICIO DE SESIÓN\n")

    intentos = 0
    max_intentos = 3

    # ── BUCLE while: repetir hasta que ingrese correctamente ──
    while intentos < max_intentos:
        usuario = input("Usuario: ").lower().strip()
        clave   = input("Clave:   ").strip()

        # ── CONDICIONAL: verificar credenciales ──
        if usuario in usuarios and usuarios[usuario]["clave"] == clave:
            print(f"\n✅ Bienvenido, {usuario.capitalize()}!\n")
            return usuario
        else:
            intentos += 1
            restantes = max_intentos - intentos
            # ── CONDICIONAL anidado: mostrar intentos restantes ──
            if restantes > 0:
                print(f"❌ Credenciales incorrectas. Intentos restantes: {restantes}\n")
            else:
                print("🚫 Acceso bloqueado. Demasiados intentos fallidos.")
                return None

    return None


def menu_principal(usuario):
    """
    Muestra el menú principal y gestiona las opciones.
    Usa un BUCLE para mantener el menú activo.
    """
    # ── BUCLE while: mantener el menú activo ──
    while True:
        print("\n" + "-"*50)
        print("           MENÚ PRINCIPAL")
        print("-"*50)
        print("  1. Registrar ingreso")
        print("  2. Registrar gasto")
        print("  3. Ver historial de movimientos")
        print("  4. Ver estado de meta")
        print("  5. Salir")
        print("-"*50)

        opcion = input("Selecciona una opción (1-5): ").strip()

        # ── CONDICIONAL: manejar cada opción del menú ──
        if opcion == "1":
            registrar_ingreso(usuario)

        elif opcion == "2":
            registrar_gasto(usuario)

        elif opcion == "3":
            ver_historial(usuario)

        elif opcion == "4":
            ver_meta(usuario)

        elif opcion == "5":
            print("\n👋 ¡Hasta luego!\n")
            break   # Sale del bucle y termina el programa

        else:
            print("⚠️  Opción no válida. Intenta de nuevo.")


def registrar_ingreso(usuario):
    """
    Registra un ingreso, calcula distribución y notifica.
    """
    print("\n── REGISTRAR INGRESO ──")

    # ── BUCLE para validar que el monto sea un número positivo ──
    while True:
        try:
            monto = float(input("Ingresa el monto recibido: $"))
            # ── CONDICIONAL: validar monto positivo ──
            if monto <= 0:
                print("❌ El monto debe ser mayor a 0.")
            else:
                break
        except ValueError:
            print("❌ Ingresa un número válido.")

    # CAPA 2 — Lógica del sistema
    ahorro, inversion, reserva = calcular_distribucion(monto)
    estado_meta = verificar_meta(usuario, ahorro)
    alerta      = generar_alerta(monto)

    # CAPA 3 — Guardar en base de datos
    registrar_movimiento(usuario, monto, "ingreso")

    # Mostrar resultado
    print("\n" + "="*50)
    print("       RESULTADO DE LA DISTRIBUCIÓN")
    print("="*50)
    print(f"  💰 Monto ingresado: ${monto:.2f}")
    print(f"  🏦 Ahorro (40%):    ${ahorro:.2f}")
    print(f"  📈 Inversión (50%): ${inversion:.2f}")
    print(f"  🔒 Reserva (10%):   ${reserva:.2f}")

    # CAPA 4 — Notificaciones
    enviar_notificaciones(usuario, ahorro, inversion, reserva, estado_meta, alerta)


def registrar_gasto(usuario):
    """
    Registra un gasto y genera alerta si supera el límite.
    """
    print("\n── REGISTRAR GASTO ──")

    # ── BUCLE para validar el monto ──
    while True:
        try:
            monto = float(input("Ingresa el monto del gasto: $"))
            if monto <= 0:
                print("❌ El monto debe ser mayor a 0.")
            else:
                break
        except ValueError:
            print("❌ Ingresa un número válido.")

    limite = float(input("¿Cuál es tu límite de gasto permitido? $"))

    # CAPA 2 — Generar alerta
    alerta = generar_alerta(monto, limite)

    # CAPA 3 — Guardar
    registrar_movimiento(usuario, monto, "gasto")

    print(f"\n✅ Gasto de ${monto:.2f} registrado.")

    # ── CONDICIONAL: mostrar alerta si existe ──
    if alerta:
        print(alerta)
    else:
        print("👍 Gasto dentro del límite permitido.")


def ver_historial(usuario):
    """
    Muestra todos los movimientos del usuario.
    Usa un BUCLE for para recorrer la lista.
    """
    print("\n── HISTORIAL DE MOVIMIENTOS ──")

    # Filtrar movimientos del usuario
    mis_movimientos = [m for m in movimientos if m["usuario"] == usuario]

    # ── CONDICIONAL: verificar si hay movimientos ──
    if not mis_movimientos:
        print("  No hay movimientos registrados aún.")
    else:
        # ── BUCLE for: recorrer e imprimir cada movimiento ──
        for i, mov in enumerate(mis_movimientos, start=1):
            print(f"  {i}. [{mov['fecha']}] {mov['tipo'].upper()} — ${mov['monto']:.2f}")


def ver_meta(usuario):
    """
    Muestra el estado actual de la meta de ahorro.
    """
    print("\n── ESTADO DE META ──")
    acumulado  = metas[usuario]["acumulado"]
    meta       = metas[usuario]["meta_ahorro"]
    porcentaje = (acumulado / meta) * 100

    print(f"  Meta:      ${meta:.2f}")
    print(f"  Acumulado: ${acumulado:.2f}")
    print(f"  Progreso:  {porcentaje:.1f}%")

    # ── BUCLE for: mostrar barra de progreso visual ──
    bloques_llenos  = int(porcentaje / 10)
    bloques_vacios  = 10 - bloques_llenos
    barra = "█" * bloques_llenos + "░" * bloques_vacios
    print(f"  [{barra}] {porcentaje:.1f}%")

    # ── CONDICIONAL: mensaje según progreso ──
    if porcentaje >= 100:
        print("  🎉 ¡Meta alcanzada!")
    elif porcentaje >= 50:
        print("  💪 ¡Vas muy bien, sigue así!")
    else:
        print("  🚀 ¡Sigue ahorrando, puedes lograrlo!")


# ─── PUNTO DE ENTRADA DEL PROGRAMA ───────────────────────────

if __name__ == "__main__":
    usuario = iniciar_sesion()

    # ── CONDICIONAL: solo continuar si el login fue exitoso ──
    if usuario:
        menu_principal(usuario)