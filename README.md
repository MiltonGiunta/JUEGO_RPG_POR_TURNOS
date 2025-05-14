# JUEGO_RPG_POR_TURNOS
# seleccionar primero el lenguaje de progamacion Pythom, despues crear un Virtual Enviroment...
# seleccionar el .vent y su version actual de Pythom, seleccionar la carpeta donde va a guardarse
# ya se puede empejar a trabajar en el proyecto
# instalar pygame en la terminal de esta manera: pip install pygame
# donde dice "nombre de su carpeta" van el nombre que le hayan puesto a sus carpetas del proyecto

    import pygame 
    import sys
    import random


    def main():
      pygame.init()
      ANCHO, ALTO = 800, 600
      pantalla = pygame.display.set_mode((ANCHO, ALTO))
      pygame.display.set_caption("Mi RPG de Turnos")
      reloj = pygame.time.Clock()
    
      imagen_mago = pygame.image.load("nombre_de_su_carpeta.py/mage.png").convert_alpha()
      imagen_guerrero = pygame.image.load("nombre_de_su_carpeta.py/warrior.png").convert_alpha()

     class Personaje:
        def __init__(self,nombre, imagen, posicion, vida_max, poder_normal, poder_especial):
            self.nombre = nombre
            self.imagen = imagen
            self.x, self.y = posicion
            self.vida_max = vida_max
            self.vida = vida_max
            self.poder_normal = poder_normal
            self.poder_especial = poder_especial
            

            

        def dibujar(self):
            pantalla.blit(self.imagen, (self.x, self.y))
            ancho_barra = self.imagen.get_width()
            alto_barra = 10
            proporción = self.vida / self.vida_max
            pygame.draw.rect(pantalla, (255, 0, 0), (self.x, self.y - 15, ancho_barra, alto_barra))
            pygame.draw.rect(pantalla, (0, 255, 0), (self.x, self.y - 15, int(ancho_barra * proporción), alto_barra))

        def recibir_daño(self, daño):
            if random.random() < 0.2:
                return False  # esquivó
            self.vida = max(0, self.vida - daño)
            return True

        def curar(self):
            curacion = 20
            self.vida = min(self.vida_max, self.vida + curacion)

    mago = Personaje("Mago", imagen_mago, (150, 300), 100, 15, 30)
    guerrero = Personaje("Guerrero", imagen_guerrero, (500, 300), 120, 10, 25)

    fuente_grande = pygame.font.SysFont(None, 36)
    fuente_pequeña = pygame.font.SysFont(None, 24)

    while True:
        mago.vida = mago.vida_max
        guerrero.vida = guerrero.vida_max

        seleccionado = False
        while not seleccionado:
            for evento in pygame.event.get():
                if evento.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif evento.type == pygame.KEYDOWN:
                    if evento.key == pygame.K_1:
                        jugador, maquina = mago, guerrero
                        seleccionado = True
                    elif evento.key == pygame.K_2:
                        jugador, maquina = guerrero, mago
                        seleccionado = True

            pantalla.fill((30, 30, 30))
            texto1 = fuente_grande.render("1 - Soy Mago", True, (255, 255, 255))
            texto2 = fuente_grande.render("2 - Soy Guerrero", True, (255, 255, 255))
            pantalla.blit(texto1, (ANCHO//2 - texto1.get_width()//2, ALTO//2 - 40))
            pantalla.blit(texto2, (ANCHO//2 - texto2.get_width()//2, ALTO//2 + 10))
            pygame.display.flip()
            reloj.tick(60)

        turno_jugador = True
        texto_accion = ""
        juego_terminado = False
        tiempo_turno = 5000  # 5 segundos por turno
        tiempo_inicio_turno = pygame.time.get_ticks()

        while not juego_terminado:
            ahora = pygame.time.get_ticks()

            for evento in pygame.event.get():
                if evento.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif evento.type == pygame.KEYDOWN and turno_jugador:
                    if evento.key == pygame.K_a:
                        exito = maquina.recibir_daño(jugador.poder_normal)
                        texto_accion = f"{jugador.nombre} usó Ataque Normal! {'Golpeó' if exito else 'Esquivó'}"
                        turno_jugador = False
                        tiempo_inicio_turno = ahora
                    elif evento.key == pygame.K_s:
                        exito = maquina.recibir_daño(jugador.poder_especial)
                        texto_accion = f"{jugador.nombre} usó Ataque Especial! {'Golpeó' if exito else 'Esquivó'}"
                        turno_jugador = False
                        tiempo_inicio_turno = ahora
                    elif evento.key == pygame.K_d:
                        jugador.curar()
                        texto_accion = f"{jugador.nombre} se curó!"
                        turno_jugador = False
                        tiempo_inicio_turno = ahora

            if turno_jugador and ahora - tiempo_inicio_turno > tiempo_turno:
                texto_accion = f"{jugador.nombre} tardó mucho... pierde el turno!"
                turno_jugador = False
                tiempo_inicio_turno = ahora

            if not turno_jugador and maquina.vida > 0 and jugador.vida > 0:
                pygame.time.delay(700)
                if random.random() < 0.5:
                    daño, nombre_a = maquina.poder_normal, 'Normal'
                else:
                    daño, nombre_a = maquina.poder_especial, 'Especial'
                exito = jugador.recibir_daño(daño)
                texto_accion = f"{maquina.nombre} usó {nombre_a}! {'Te golpeó' if exito else 'Esquivaste'}"
                turno_jugador = True
                tiempo_inicio_turno = pygame.time.get_ticks()

            pantalla.fill((30, 30, 30))
            mago.dibujar()
            guerrero.dibujar()

            pantalla.blit(fuente_pequeña.render(texto_accion, True, (255, 255, 0)), (20, 5))
            turno_txt = f"Turno: {jugador.nombre if turno_jugador else maquina.nombre}"
            pantalla.blit(fuente_pequeña.render(turno_txt, True, (255, 255, 255)), (20, 30))
            if turno_jugador:
                opciones = fuente_pequeña.render("A=Normal  S=Especial  D=Curar (5s para actuar)", True, (200, 200, 200))
                pantalla.blit(opciones, (20, 55))

            if maquina.vida == 0 or jugador.vida == 0:
                juego_terminado = True
                ganador = jugador.nombre if maquina.vida == 0 else maquina.nombre

            pygame.display.flip()
            reloj.tick(60)

        final = True
        while final:
            for evento in pygame.event.get():
                if evento.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif evento.type == pygame.KEYDOWN:
                    if evento.key == pygame.K_r:
                        final = False
                    elif evento.key == pygame.K_q:
                        pygame.quit()
                        sys.exit()

            pantalla.fill((30, 30, 30))
            fin_text = fuente_grande.render(f"¡{ganador} ganó! R=Reintentar Q=Salir", True, (0, 255, 0))
            pantalla.blit(fin_text, (ANCHO//2 - fin_text.get_width()//2, ALTO//2 - fin_text.get_height()//2))
            pygame.display.flip()
            reloj.tick(60)

    if __name__ == "__main__":
    main()
