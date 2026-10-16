[insert.py](https://github.com/user-attachments/files/33525751/insert.py)
# Python
import math
import random
import sys
import pygame


# ============================================================
# PREMIUM LOVE
# Professional Heart Animation
#
# Install:
#     pip install pygame
#
# Run:
#     python premium_love.py
#
# Controls:
#     LEFT CLICK = Love Burst
#     SPACE      = Love Burst
#     ESC        = Exit
# ============================================================


# ============================================================
# INITIALIZATION
# ============================================================

pygame.init()

WIDTH = 1100
HEIGHT = 720
FPS = 60

screen = pygame.display.set_mode(
    (WIDTH, HEIGHT),
    pygame.SCALED | pygame.RESIZABLE
)

pygame.display.set_caption("💗 LOVE — warn")

clock = pygame.time.Clock()


# ============================================================
# COLORS
# ============================================================

BG_TOP = (25, 8, 28)
BG_BOTTOM = (91, 16, 58)

HEART_DARK = (145, 8, 50)
HEART_MAIN = (255, 30, 85)
HEART_LIGHT = (255, 100, 150)

WHITE = (255, 245, 250)

PINK_1 = (255, 55, 110)
PINK_2 = (255, 100, 150)
PINK_3 = (255, 165, 195)
PINK_4 = (255, 220, 235)


# ============================================================
# GLOBAL CENTER
# ============================================================

CENTER = pygame.Vector2(WIDTH / 2, HEIGHT / 2)


# ============================================================
# FONT SYSTEM
# ============================================================

def get_font(size, bold=False):

    candidates = [
        "tahoma",
        "segoeui",
        "arial",
        "dejavusans"
    ]

    for name in candidates:

        path = pygame.font.match_font(
            name,
            bold=bold
        )

        if path:
            return pygame.font.Font(
                path,
                size
            )

    return pygame.font.Font(
        None,
        size
    )


FONT_TITLE = get_font(76, True)
FONT_HINT = get_font(18, False)


# ============================================================
# MATH UTILITIES
# ============================================================

def clamp(value, minimum=0.0, maximum=1.0):

    return max(
        minimum,
        min(maximum, value)
    )


def ease_out_cubic(t):

    t = clamp(t)

    return 1 - (1 - t) ** 3


def ease_in_out_sine(t):

    t = clamp(t)

    return -(math.cos(math.pi * t) - 1) / 2


def lerp(a, b, t):

    return a + (b - a) * t


# ============================================================
# HEART MATHEMATICS
# ============================================================

def heart_points(
    center,
    scale,
    samples=220
):

    points = []

    cx = center.x
    cy = center.y

    for i in range(samples + 1):

        t = (
            i / samples
        ) * math.tau

        x = (
            16 *
            math.sin(t) ** 3
        )

        y = (
            13 * math.cos(t)
            - 5 * math.cos(2 * t)
            - 2 * math.cos(3 * t)
            - math.cos(4 * t)
        )

        px = cx + x * scale
        py = cy - y * scale

        points.append(
            (px, py)
        )

    return points


# ============================================================
# BACKGROUND
# ============================================================

def create_gradient(
    width,
    height
):

    surface = pygame.Surface(
        (width, height)
    )

    for y in range(height):

        t = y / max(
            1,
            height - 1
        )

        t = t * t

        r = int(
            lerp(
                BG_TOP[0],
                BG_BOTTOM[0],
                t
            )
        )

        g = int(
            lerp(
                BG_TOP[1],
                BG_BOTTOM[1],
                t
            )
        )

        b = int(
            lerp(
                BG_TOP[2],
                BG_BOTTOM[2],
                t
            )
        )

        pygame.draw.line(
            surface,
            (r, g, b),
            (0, y),
            (width, y)
        )

    return surface


# ============================================================
# GLOW SYSTEM
# ============================================================

class GlowRenderer:

    def __init__(self):

        self.cache = {}


    def create_glow(
        self,
        radius,
        color,
        alpha
    ):

        radius = int(radius)
        alpha = int(alpha)

        key = (
            radius,
            color,
            alpha
        )

        if key in self.cache:

            return self.cache[key]

        size = (
            radius * 2 +
            20
        )

        surface = pygame.Surface(
            (size, size),
            pygame.SRCALPHA
        )

        for i in range(10, 0, -1):

            current_radius = int(
                radius *
                i /
                10
            )

            current_alpha = int(
                alpha *
                (1 - i / 11) ** 1.7
            )

            if current_radius <= 0:
                continue

            if current_alpha <= 0:
                continue

            pygame.draw.circle(
                surface,
                (
                    color[0],
                    color[1],
                    color[2],
                    current_alpha
                ),
                (
                    size // 2,
                    size // 2
                ),
                current_radius
            )

        self.cache[key] = surface

        return surface


    def draw(
        self,
        target,
        position,
        radius,
        color,
        alpha
    ):

        glow = self.create_glow(
            radius,
            color,
            alpha
        )

        rect = glow.get_rect(
            center=(
                int(position.x),
                int(position.y)
            )
        )

        target.blit(
            glow,
            rect
        )


glow = GlowRenderer()


# ============================================================
# HEART PARTICLE
# ============================================================

class HeartParticle:

    def __init__(
        self,
        position,
        velocity,
        size,
        color,
        lifetime
    ):

        self.position = pygame.Vector2(
            position
        )

        self.velocity = pygame.Vector2(
            velocity
        )

        self.size = size

        self.color = color

        self.lifetime = lifetime

        self.max_lifetime = lifetime

        self.rotation = random.uniform(
            0,
            360
        )

        self.rotation_speed = random.uniform(
            -220,
            220
        )

        self.gravity = random.uniform(
            10,
            40
        )


    @property
    def alive(self):

        return self.lifetime > 0


    def update(self, dt):

        self.lifetime -= dt

        self.velocity.y += (
            self.gravity *
            dt
        )

        self.position += (
            self.velocity *
            dt
        )

        self.rotation += (
            self.rotation_speed *
            dt
        )


    def draw(self, target):

        if not self.alive:
            return

        life_ratio = clamp(
            self.lifetime /
            self.max_lifetime
        )

        alpha = int(
            255 *
            life_ratio
        )

        size = max(
            3,
            int(self.size)
        )

        surface_size = size * 5

        surface = pygame.Surface(
            (
                surface_size,
                surface_size
            ),
            pygame.SRCALPHA
        )

        center = pygame.Vector2(
            surface_size / 2,
            surface_size / 2
        )

        points = heart_points(
            center,
            size / 18,
            80
        )

        pygame.draw.polygon(
            surface,
            (
                self.color[0],
                self.color[1],
                self.color[2],
                alpha
            ),
            points
        )

        rotated = pygame.transform.rotate(
            surface,
            self.rotation
        )

        rect = rotated.get_rect(
            center=(
                int(self.position.x),
                int(self.position.y)
            )
        )

        target.blit(
            rotated,
            rect
        )


# ============================================================
# SPARK PARTICLE
# ============================================================

class SparkParticle:

    def __init__(
        self,
        position,
        velocity,
        size,
        lifetime
    ):

        self.position = pygame.Vector2(
            position
        )

        self.velocity = pygame.Vector2(
            velocity
        )

        self.size = size

        self.lifetime = lifetime

        self.max_lifetime = lifetime


    @property
    def alive(self):

        return self.lifetime > 0


    def update(self, dt):

        self.lifetime -= dt

        self.position += (
            self.velocity *
            dt
        )

        self.velocity *= (
            1 -
            dt * 1.4
        )


    def draw(self, target):

        if not self.alive:
            return

        ratio = clamp(
            self.lifetime /
            self.max_lifetime
        )

        alpha = int(
            255 *
            ratio
        )

        length = max(
            2,
            self.size * 2
        )

        if self.velocity.length() > 0:

            direction = (
                self.velocity.normalize()
            )

        else:

            direction = pygame.Vector2(
                1,
                0
            )

        end = (
            self.position +
            direction * length
        )

        pygame.draw.line(
            target,
            (
                255,
                235,
                245,
                alpha
            ),
            self.position,
            end,
            max(
                1,
                int(self.size)
            )
        )


# ============================================================
# RIPPLE EFFECT
# ============================================================

class Ripple:

    def __init__(self, position):

        self.position = pygame.Vector2(
            position
        )

        self.time = 0

        self.duration = 0.75


    @property
    def alive(self):

        return (
            self.time <
            self.duration
        )


    def update(self, dt):

        self.time += dt


    def draw(self, target):

        if not self.alive:
            return

        progress = clamp(
            self.time /
            self.duration
        )

        radius = lerp(
            20,
            280,
            ease_out_cubic(progress)
        )

        alpha = int(
            150 *
            (1 - progress)
        )

        width = max(
            1,
            int(
                5 *
                (1 - progress)
            )
        )

        pygame.draw.circle(
            target,
            (
                255,
                140,
                180,
                alpha
            ),
            (
                int(self.position.x),
                int(self.position.y)
            ),
            int(radius),
            width
        )


# ============================================================
# AMBIENT FLOATING HEART
# ============================================================

class FloatingHeart:

    def __init__(
        self,
        width,
        height
    ):

        self.position = pygame.Vector2(
            random.uniform(
                20,
                width - 20
            ),
            height + random.uniform(
                20,
                100
            )
        )

        self.speed = random.uniform(
            20,
            70
        )

        self.wave = random.uniform(
            7,
            28
        )

        self.phase = random.uniform(
            0,
            math.tau
        )

        self.size = random.uniform(
            5,
            14
        )

        self.time = 0

        self.max_time = random.uniform(
            8,
            15
        )

        self.color = random.choice(
            [
                PINK_1,
                PINK_2,
                PINK_3,
                PINK_4
            ]
        )


    @property
    def alive(self):

        return (
            self.time <
            self.max_time
            and
            self.position.y > -80
        )


    def update(self, dt):

        self.time += dt

        self.position.y -= (
            self.speed *
            dt
        )

        self.position.x += (
            math.sin(
                self.time * 1.5 +
                self.phase
            )
            *
            self.wave *
            dt
        )


    def draw(self, target):

        if not self.alive:
            return

        fade_in = min(
            1,
            self.time
        )

        fade_out = min(
            1,
            (
                self.max_time -
                self.time
            ) / 2
        )

        alpha = int(
            80 *
            min(
                fade_in,
                fade_out
            )
        )

        if alpha <= 0:
            return

        size = max(
            3,
            int(self.size)
        )

        surface = pygame.Surface(
            (
                80,
                80
            ),
            pygame.SRCALPHA
        )

        points = heart_points(
            pygame.Vector2(
                40,
                40
            ),
            size / 18,
            70
        )

        pygame.draw.polygon(
            surface,
            (
                self.color[0],
                self.color[1],
                self.color[2],
                alpha
            ),
            points
        )

        target.blit(
            surface,
            surface.get_rect(
                center=(
                    int(self.position.x),
                    int(self.position.y)
                )
            )
        )


# ============================================================
# TEXT ANIMATION
# ============================================================

class LoveText:

    def __init__(self):

        self.visible = False

        self.time = 0

        self.duration = 0.75


    def trigger(self):

        self.visible = True

        self.time = 0


    def update(self, dt):

        if not self.visible:
            return

        self.time += dt


    def draw(
        self,
        target,
        position
    ):

        if not self.visible:
            return

        progress = clamp(
            self.time /
            self.duration
        )

        scale = lerp(
            0.55,
            1.0,
            ease_out_cubic(progress)
        )

        alpha = int(
            255 *
            ease_out_cubic(progress)
        )

        text = FONT_TITLE.render(
            "หวาน",
            True,
            WHITE
        )

        text = text.convert_alpha()

        scaled = pygame.transform.smoothscale(
            text,
            (
                max(
                    1,
                    int(
                        text.get_width() *
                        scale
                    )
                ),
                max(
                    1,
                    int(
                        text.get_height() *
                        scale
                    )
                )
            )
        )

        scaled.set_alpha(
            alpha
        )

        # Glow behind text
        glow_surface = pygame.Surface(
            (
                scaled.get_width() + 60,
                scaled.get_height() + 60
            ),
            pygame.SRCALPHA
        )

        glow_text = FONT_TITLE.render(
            "หวาน",
            True,
            HEART_LIGHT
        )

        glow_text = pygame.transform.smoothscale(
            glow_text,
            scaled.get_size()
        )

        for radius, a in [
            (12, 25),
            (7, 40),
            (3, 60)
        ]:

            temp = glow_text.copy()

            temp.set_alpha(
                int(
                    a *
                    alpha /
                    255
                )
            )

            glow_surface.blit(
                temp,
                (
                    (
                        glow_surface.get_width()
                        -
                        temp.get_width()
                    ) // 2,
                    (
                        glow_surface.get_height()
                        -
                        temp.get_height()
                    ) // 2
                    + radius
                )
            )

        rect = scaled.get_rect(
            center=(
                int(position.x),
                int(position.y - 5)
            )
        )

        glow_rect = glow_surface.get_rect(
            center=rect.center
        )

        target.blit(
            glow_surface,
            glow_rect
        )

        target.blit(
            scaled,
            rect
        )


# ============================================================
# MAIN APPLICATION
# ============================================================

class LoveApp:

    def __init__(self):

        self.running = True

        self.particles = []

        self.ripples = []

        self.floating_hearts = []

        self.love_text = LoveText()

        self.time = 0

        self.pulse_time = 0

        self.click_boost = 0

        self.spawn_timer = 0

        self.background = create_gradient(
            WIDTH,
            HEIGHT
        )


    # --------------------------------------------------------
    # Resize
    # --------------------------------------------------------

    def resize(
        self,
        size
    ):

        global WIDTH
        global HEIGHT
        global CENTER

        WIDTH = max(
            700,
            size[0]
        )

        HEIGHT = max(
            500,
            size[1]
        )

        CENTER = pygame.Vector2(
            WIDTH / 2,
            HEIGHT / 2
        )

        self.background = create_gradient(
            WIDTH,
            HEIGHT
        )


    # --------------------------------------------------------
    # Heartbeat
    # --------------------------------------------------------

    def get_heartbeat_scale(self):

        cycle = (
            self.pulse_time %
            1.35
        )

        beat = 0

        # First beat
        if cycle < 0.16:

            t = (
                cycle /
                0.16
            )

            beat = math.sin(
                math.pi * t
            )

        # Second smaller beat
        elif cycle < 0.34:

            t = (
                (
                    cycle -
                    0.16
                )
                /
                0.18
            )

            beat = (
                math.sin(
                    math.pi * t
                )
                *
                0.55
            )

        return (
            8.6
            +
            beat * 0.95
            +
            self.click_boost
        )


    # --------------------------------------------------------
    # Ambient Spawn
    # --------------------------------------------------------

    def spawn_ambient_heart(self):

        self.floating_hearts.append(
            FloatingHeart(
                WIDTH,
                HEIGHT
            )
        )


    # --------------------------------------------------------
    # Love Burst
    # --------------------------------------------------------

    def create_love_burst(
        self,
        amount=75
    ):

        self.ripples.append(
            Ripple(
                CENTER
            )
        )

        # Main heart particles
        for _ in range(amount):

            angle = random.uniform(
                0,
                math.tau
            )

            speed = random.uniform(
                90,
                380
            )

            velocity = pygame.Vector2(
                math.cos(angle),
                math.sin(angle)
            ) * speed

            color = random.choice(
                [
                    PINK_1,
                    PINK_2,
                    PINK_3,
                    PINK_4,
                    WHITE
                ]
            )

            self.particles.append(
                HeartParticle(
                    CENTER,
                    velocity,
                    random.uniform(
                        7,
                        20
                    ),
                    color,
                    random.uniform(
                        0.8,
                        1.8
                    )
                )
            )

        # Spark explosion
        for _ in range(30):

            angle = random.uniform(
                0,
                math.tau
            )

            speed = random.uniform(
                180,
                550
            )

            velocity = pygame.Vector2(
                math.cos(angle),
                math.sin(angle)
            ) * speed

            self.particles.append(
                SparkParticle(
                    CENTER,
                    velocity,
                    random.uniform(
                        2,
                        5
                    ),
                    random.uniform(
                        0.3,
                        0.8
                    )
                )
            )


    # --------------------------------------------------------
    # Click
    # --------------------------------------------------------

    def click_heart(self):

        self.love_text.trigger()

        self.click_boost = 3.5

        self.create_love_burst(
            85
        )


    # --------------------------------------------------------
    # Update
    # --------------------------------------------------------

    def update(
        self,
        dt
    ):

        self.time += dt

        self.pulse_time += dt

        self.click_boost = max(
            0,
            self.click_boost -
            dt * 9
        )

        # Spawn floating hearts
        self.spawn_timer += dt

        if self.spawn_timer >= random.uniform(
            0.25,
            0.55
        ):

            self.spawn_timer = 0

            self.spawn_ambient_heart()


        # Particles
        for particle in self.particles:

            particle.update(
                dt
            )

        self.particles = [
            p
            for p in self.particles
            if p.alive
        ]


        # Ripples
        for ripple in self.ripples:

            ripple.update(
                dt
            )

        self.ripples = [
            r
            for r in self.ripples
            if r.alive
        ]


        # Ambient hearts
        for heart in self.floating_hearts:

            heart.update(
                dt
            )

        self.floating_hearts = [
            h
            for h in self.floating_hearts
            if h.alive
        ]


        self.love_text.update(
            dt
        )


    # --------------------------------------------------------
    # Background Effects
    # --------------------------------------------------------

    def draw_background(
        self,
        target
    ):

        target.blit(
            self.background,
            (
                0,
                0
            )
        )

        # Large atmospheric glow
        glow.draw(
            target,
            CENTER,
            300,
            (
                255,
                20,
                90
            ),
            25
        )

        # Fixed subtle particles
        random.seed(
            2026
        )

        for _ in range(110):

            x = random.randrange(
                WIDTH
            )

            y = random.randrange(
                HEIGHT
            )

            size = random.choice(
                [
                    1,
                    1,
                    1,
                    2
                ]
            )

            pygame.draw.circle(
                target,
                (
                    255,
                    190,
                    215,
                    35
                ),
                (
                    x,
                    y
                ),
                size
            )

        random.seed()


    # --------------------------------------------------------
    # Main Heart
    # --------------------------------------------------------

    def draw_main_heart(
        self,
        target
    ):

        scale = (
            self.get_heartbeat_scale()
        )

        # Outer glow
        glow.draw(
            target,
            CENTER,
            175 + scale * 4,
            (
                255,
                20,
                90
            ),
            30
        )

        # Inner glow
        glow.draw(
            target,
            CENTER,
            115 + scale * 3,
            (
                255,
                60,
                120
            ),
            50
        )

        # Shadow / outer heart
        outer = heart_points(
            CENTER,
            scale * 1.08
        )

        pygame.draw.polygon(
            target,
            HEART_DARK,
            outer
        )

        # Main body
        main = heart_points(
            CENTER,
            scale
        )

        pygame.draw.polygon(
            target,
            HEART_MAIN,
            main
        )

        # Highlight
        overlay = pygame.Surface(
            (
                WIDTH,
                HEIGHT
            ),
            pygame.SRCALPHA
        )

        highlight_center = (
            CENTER +
            pygame.Vector2(
                -1.5,
                -3
            )
        )

        highlight = heart_points(
            highlight_center,
            scale * 0.86
        )

        pygame.draw.polygon(
            overlay,
            (
                HEART_LIGHT[0],
                HEART_LIGHT[1],
                HEART_LIGHT[2],
                95
            ),
            highlight
        )

        target.blit(
            overlay,
            (
                0,
                0
            )
        )

        # Outline
        pygame.draw.lines(
            target,
            (
                255,
                160,
                190
            ),
            True,
            main,
            2
        )


    # --------------------------------------------------------
    # Bottom UI
    # --------------------------------------------------------

    def draw_hint(
        self,
        target
    ):

        text = FONT_HINT.render(
            "CLICK THE HEART  •  LOVE",
            True,
            (
                255,
                220,
                235
            )
        )

        text.set_alpha(
            150
        )

        rect = text.get_rect(
            center=(
                WIDTH // 2,
                HEIGHT - 35
            )
        )

        target.blit(
            text,
            rect
        )


    # --------------------------------------------------------
    # Render
    # --------------------------------------------------------

    def draw(self):

        self.draw_background(
            screen
        )

        # Background hearts
        for heart in self.floating_hearts:

            heart.draw(
                screen
            )

        # Ripple behind heart
        for ripple in self.ripples:

            ripple.draw(
                screen
            )

        # Main heart
        self.draw_main_heart(
            screen
        )

        # Explosion particles
        for particle in self.particles:

            particle.draw(
                screen
            )

        # Text
        self.love_text.draw(
            screen,
            CENTER
        )

        self.draw_hint(
            screen
        )

        pygame.display.flip()


    # --------------------------------------------------------
    # Main Loop
    # --------------------------------------------------------

    def run(self):

        while self.running:

            dt = (
                clock.tick(FPS)
                /
                1000.0
            )

            # Prevent huge jumps if the window freezes.
            dt = min(
                dt,
                0.05
            )

            for event in pygame.event.get():

                # Close
                if event.type == pygame.QUIT:

                    self.running = False


                # Resize
                elif event.type == pygame.VIDEORESIZE:

                    self.resize(
                        event.size
                    )


                # Keyboard
                elif event.type == pygame.KEYDOWN:

                    if event.key == pygame.K_ESCAPE:

                        self.running = False

                    elif event.key == pygame.K_SPACE:

                        self.click_heart()


                # Mouse
                elif event.type == pygame.MOUSEBUTTONDOWN:

                    if event.button == 1:

                        mouse = pygame.Vector2(
                            event.pos
                        )

                        distance = (
                            mouse -
                            CENTER
                        ).length()

                        # Generous professional hitbox
                        if distance < 145:

                            self.click_heart()


            self.update(
                dt
            )

            self.draw()


        pygame.quit()


# ============================================================
# PROGRAM ENTRY
# ============================================================

if __name__ == "__main__":

    try:

        LoveApp().run()

    except Exception as error:

        pygame.quit()

        print()
        print("=" * 60)
        print("LOVE PROGRAM ERROR")
        print("=" * 60)
        print(error)
        print("=" * 60)

        sys.exit(1)
