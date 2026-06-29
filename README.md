# ─── CAPA 3: BASE DE DATOS (simulada con diccionarios) ───────
usuarios = {}
movimientos = []
metas = {}

# ─── CAPA 2: LÓGICA DEL SISTEMA ──────────────────────────────

def calcular_distribucion(monto, pct_ahorro, pct_inversion, pct_reserva):
    ahorro    = monto * (pct_ahorro / 100)
    inversion = monto * (pct_inversion / 100)
    reserva   = monto * (pct_reserva / 100)
    return ahorro, inversion, reserva

def verificar_meta(usuario, nuevo_ahorro):
    metas[usuario]["acumulado"] += nuevo_ahorro
    acumulado  = metas[usuario]["acumulado"]
    meta       = metas[usuario]["meta_ahorro"]
    porcentaje = (acumulado / meta) * 100
    if acumulado >= meta:
        return f"🎉 ¡Felicidades! Alcanzaste tu meta de ${meta:.2f}"
    else:
        return f"📊 Meta al {porcentaje:.1f}% — Te faltan ${meta - acumulado:.2f}"

def generar_alerta(monto, limite):
    if monto > limite:
        return f"⚠️  ALERTA: Gasto de ${monto:.2f} supera tu límite de ${limite:.2f}"
    return None

def registrar_movimiento(usuario, monto, tipo):
    from datetime import date
    movimientos.append({
        "usuario": usuario,
        "monto":   monto,
        "fecha":   str(date.today()),
        "tipo":    tipo
    })

# ─── CAPA 4: NOTIFICACIONES ───────────────────────────────────

def enviar_notificaciones(usuario, ahorro, inversion, reserva, estado_meta, alerta):
    print("\n" + "="*50)
    print("       CAPA 4 — NOTIFICACIONES")
    print("="*50)
    print(f"\n📧 Email enviado a {usuarios[usuario]['email']}:")
    print(f"   'Guardaste ${ahorro:.2f} hoy'")
    print(f"\n🖥️  Alerta en pantalla:")
    print(f"   '{estado_meta}'")
    print(f"\n📄 Reporte generado:")
    pcts = usuarios[usuario]["porcentajes"]
    print(f"   Ahorro ({pcts[0]}%):    ${ahorro:.2f}")
    print(f"   Inversión ({pcts[1]}%): ${inversion:.2f}")
    print(f"   Reserva ({pcts[2]}%):   ${reserva:.2f}")
    if alerta:
        print(f"\n{alerta}")

# ─── CAPA 1: INTERFAZ DE USUARIO ─────────────────────────────

def ingresar_numero(mensaje, positivo=True):
    while True:
        try:
            valor = float(input(mensaje))
            if positivo and valor <= 0:
                print("❌ El valor debe ser mayor a 0.")
            else:
                return valor
        except ValueError:
            print("❌ Ingresa un número válido.")

def registrar_nuevo_usuario():
    print("\n── CREAR NUEVO USUARIO ──")
    while True:
        nombre = input("Nombre de usuario: ").lower().strip()
        if not nombre:
            print("❌ El nombre no puede estar vacío.")
        elif nombre in usuarios:
            print("❌ Ese usuario ya existe.")
        else:
            break

    email = input("Email: ").strip()

    while True:
        clave = input("Clave: ").strip()
        if len(clave) < 4:
            print("❌ La clave debe tener al menos 4 caracteres.")
        else:
            break

    meta = ingresar_numero("Meta de ahorro ($): ")
    limite = ingresar_numero("Límite de gasto mensual ($): ")

    print("\nDefine tu distribución de ingresos (deben sumar 100%):")
    while True:
        pa = ingresar_numero("  % Ahorro: ")
        pi = ingresar_numero("  % Inversión: ")
        pr = ingresar_numero("  % Reserva: ")
        if abs(pa + pi + pr - 100) < 0.01:
            break
        print(f"❌ Suman {pa+pi+pr:.1f}%. Deben sumar exactamente 100%.")

    usuarios[nombre] = {
        "email":       email,
        "clave":       clave,
        "limite":      limite,
        "porcentajes": [pa, pi, pr]
    }
    metas[nombre] = {"meta_ahorro": meta, "acumulado": 0}
    print(f"\n✅ Usuario '{nombre}' creado exitosamente.")

def iniciar_sesion():
    print("\n" + "="*50)
    print("    SISTEMA DE GESTIÓN FINANCIERA PERSONAL v2.0")
    print("="*50)
    print("\n🔐 INICIO DE SESIÓN\n")

    intentos = 0
    while intentos < 3:
        usuario = input("Usuario: ").lower().strip()
        clave   = input("Clave:   ").strip()
        if usuario in usuarios and usuarios[usuario]["clave"] == clave:
            print(f"\n✅ Bienvenido, {usuario.capitalize()}!\n")
            return usuario
        else:
            intentos += 1
            restantes = 3 - intentos
            if restantes > 0:
                print(f"❌ Credenciales incorrectas. Intentos restantes: {restantes}\n")
            else:
                print("🚫 Acceso bloqueado. Demasiados intentos fallidos.")
                return None
    return None

def menu_principal(usuario):
    while True:
        pcts = usuarios[usuario]["porcentajes"]
        limite = usuarios[usuario]["limite"]
        print("\n" + "-"*50)
        print(f"  Usuario: {usuario.capitalize()}")
        print(f"  Distribución: {pcts[0]}% ahorro | {pcts[1]}% inversión | {pcts[2]}% reserva")
        print(f"  Límite de gasto: ${limite:.2f}")
        print("-"*50)
        print("  1. Registrar ingreso")
        print("  2. Registrar gasto")
        print("  3. Ver historial de movimientos")
        print("  4. Ver estado de meta")
        print("  5. Configurar mi perfil")
        print("  6. Salir")
        print("-"*50)
        opcion = input("Selecciona una opción (1-6): ").strip()

        if opcion == "1":
            registrar_ingreso(usuario)
        elif opcion == "2":
            registrar_gasto(usuario)
        elif opcion == "3":
            ver_historial(usuario)
        elif opcion == "4":
            ver_meta(usuario)
        elif opcion == "5":
            configurar_perfil(usuario)
        elif opcion == "6":
            print("\n👋 ¡Hasta luego!\n")
            break
        else:
            print("⚠️  Opción no válida.")


def registrar_ingreso(usuario):
    print("\n── REGISTRAR INGRESO ──")
    monto = ingresar_numero("Ingresa el monto recibido: $")
    pcts  = usuarios[usuario]["porcentajes"]
    ahorro, inversion, reserva = calcular_distribucion(monto, pcts[0], pcts[1], pcts[2])
    estado_meta = verificar_meta(usuario, ahorro)
    alerta      = generar_alerta(monto, usuarios[usuario]["limite"])
    registrar_movimiento(usuario, monto, "ingreso")

    print("\n" + "="*50)
    print("       RESULTADO DE LA DISTRIBUCIÓN")
    print("="*50)
    print(f"  💰 Monto ingresado: ${monto:.2f}")
    print(f"  🏦 Ahorro ({pcts[0]}%):    ${ahorro:.2f}")
    print(f"  📈 Inversión ({pcts[1]}%): ${inversion:.2f}")
    print(f"  🔒 Reserva ({pcts[2]}%):   ${reserva:.2f}")
    enviar_notificaciones(usuario, ahorro, inversion, reserva, estado_meta, alerta)


def registrar_gasto(usuario):
    print("\n── REGISTRAR GASTO ──")
    monto  = ingresar_numero("Ingresa el monto del gasto: $")
    limite = usuarios[usuario]["limite"]
    alerta = generar_alerta(monto, limite)
    registrar_movimiento(usuario, monto, "gasto")
    print(f"\n✅ Gasto de ${monto:.2f} registrado.")
    if alerta:
        print(alerta)
    else:
        print("👍 Gasto dentro de tu límite permitido.")

def ver_historial(usuario):
    print("\n── HISTORIAL DE MOVIMIENTOS ──")
    mis_movimientos = [m for m in movimientos if m["usuario"] == usuario]
    if not mis_movimientos:
        print("  No hay movimientos registrados aún.")
    else:
        total_ingresos = 0
        total_gastos   = 0
        for i, mov in enumerate(mis_movimientos, start=1):
            print(f"  {i}. [{mov['fecha']}] {mov['tipo'].upper()} — ${mov['monto']:.2f}")
            if mov["tipo"] == "ingreso":
                total_ingresos += mov["monto"]
            else:
                total_gastos += mov["monto"]
        print(f"\n  Total ingresos: ${total_ingresos:.2f}")
        print(f"  Total gastos:   ${total_gastos:.2f}")
        print(f"  Balance:        ${total_ingresos - total_gastos:.2f}")


def ver_meta(usuario):
    print("\n── ESTADO DE META ──")
    acumulado  = metas[usuario]["acumulado"]
    meta       = metas[usuario]["meta_ahorro"]
    porcentaje = (acumulado / meta) * 100
    print(f"  Meta:      ${meta:.2f}")
    print(f"  Acumulado: ${acumulado:.2f}")
    bloques_llenos = int(min(porcentaje, 100) / 10)
    barra = "█" * bloques_llenos + "░" * (10 - bloques_llenos)
    print(f"  [{barra}] {porcentaje:.1f}%")
    if porcentaje >= 100:
        print("  🎉 ¡Meta alcanzada!")
    elif porcentaje >= 50:
        print("  💪 ¡Vas muy bien, sigue así!")
    else:
        print("  🚀 ¡Sigue ahorrando, puedes lograrlo!")

def configurar_perfil(usuario):
    print("\n── CONFIGURAR PERFIL ──")
    print("  1. Cambiar meta de ahorro")
    print("  2. Cambiar límite de gasto")
    print("  3. Cambiar porcentajes de distribución")
    print("  4. Volver")
    opcion = input("Selecciona una opción (1-4): ").strip()

    if opcion == "1":
        nueva_meta = ingresar_numero("Nueva meta de ahorro ($): ")
        metas[usuario]["meta_ahorro"] = nueva_meta
        print(f"✅ Meta actualizada a ${nueva_meta:.2f}")

    elif opcion == "2":
        nuevo_limite = ingresar_numero("Nuevo límite de gasto ($): ")
        usuarios[usuario]["limite"] = nuevo_limite
        print(f"✅ Límite actualizado a ${nuevo_limite:.2f}")

    elif opcion == "3":
        print("Define los nuevos porcentajes (deben sumar 100%):")
        while True:
            pa = ingresar_numero("  % Ahorro: ")
            pi = ingresar_numero("  % Inversión: ")
            pr = ingresar_numero("  % Reserva: ")
            if abs(pa + pi + pr - 100) < 0.01:
                usuarios[usuario]["porcentajes"] = [pa, pi, pr]
                print("✅ Porcentajes actualizados.")
                break
            print(f"❌ Suman {pa+pi+pr:.1f}%. Deben sumar exactamente 100%.")

    elif opcion == "4":
        return
    else:
        print("⚠️  Opción no válida.")


# ─── PUNTO DE ENTRADA ────────────────────────────────────────

if __name__ == "__main__":
    print("\n" + "="*50)
    print("    SISTEMA DE GESTIÓN FINANCIERA PERSONAL v2.0")
    print("="*50)
    print("\n¿Qué deseas hacer?")
    print("  1. Iniciar sesión")
    print("  2. Crear nuevo usuario")
    print("  3. Salir")

    while True:
        opcion = input("\nSelecciona una opción (1-3): ").strip()
        if opcion == "1":
            if not usuarios:
                print("⚠️  No hay usuarios registrados. Crea uno primero.")
            else:
                usuario = iniciar_sesion()
                if usuario:
                    menu_principal(usuario)
                break
        elif opcion == "2":
            registrar_nuevo_usuario()
            usuario = iniciar_sesion()
            if usuario:
                menu_principal(usuario)
            break
        elif opcion == "3":
            print("\n👋 ¡Hasta luego!\n")
            break
        else:
            print("⚠️  Opción no válida.")
