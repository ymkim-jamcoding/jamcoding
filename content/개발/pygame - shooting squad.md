---
description:
aliases:
created: 2026-04-03
modified: 2026-04-03
---

```python
from __future__ import annotations

import math
import random
from dataclasses import dataclass

import pygame


WIDTH, HEIGHT = 960, 640
FPS = 60

ROAD_DEPTH = 120.0
PLAYER_DIST = 8.0
ROAD_BOTTOM_Y = HEIGHT + 28
HORIZON_Y = 108
ROAD_TOP_HALF = 92
ROAD_BOTTOM_HALF = 320
ROAD_CENTER_X = WIDTH * 0.5

LANE_COUNT = 5
LANE_SPAN = (LANE_COUNT - 1) / 2
MAX_SQUAD = 48
BOSS_TRIGGER_DISTANCE = 450.0

SKY_TOP = (68, 145, 255)
SKY_BOTTOM = (184, 228, 255)
SAND_NEAR = (237, 200, 124)
SAND_FAR = (245, 221, 164)
ROAD_COLOR = (63, 66, 77)
ROAD_EDGE = (151, 118, 90)
LANE_COLOR = (245, 245, 245)
PLAYER_BLUE = (57, 140, 255)
PLAYER_RING = (84, 255, 120)
ENEMY_RED = (255, 97, 88)
ENEMY_DARK = (140, 44, 48)
BULLET_GOLD = (255, 198, 88)
BOSS_PURPLE = (159, 92, 255)
BOSS_DARK = (62, 27, 122)
TEXT_DARK = (24, 20, 34)
WHITE = (255, 255, 255)
GOOD_GREEN = (114, 255, 154)
WARN_ORANGE = (255, 162, 90)
BAD_RED = (255, 102, 102)
GATE_LEFT = (255, 110, 96, 128)
GATE_RIGHT = (104, 196, 255, 132)


def clamp(value: float, low: float, high: float) -> float:
    return max(low, min(high, value))


def lerp(a: float, b: float, t: float) -> float:
    return a + (b - a) * t


def blend(color_a: tuple[int, int, int], color_b: tuple[int, int, int], t: float) -> tuple[int, int, int]:
    return (
        int(lerp(color_a[0], color_b[0], t)),
        int(lerp(color_a[1], color_b[1], t)),
        int(lerp(color_a[2], color_b[2], t)),
    )


def format_operation(operation: tuple[str, int]) -> str:
    kind, value = operation
    if kind == "*":
        return f"x{value}"
    if kind == "/":
        return f"/{value}"
    return f"{kind}{value}"


@dataclass
class Bullet:
    lane: float
    dist: float
    speed: float
    damage: int = 1


@dataclass
class Enemy:
    lane: float
    dist: float
    hp: int
    speed: float
    kind: str
    wobble: float
    reward: int


@dataclass
class GatePair:
    dist: float
    left: tuple[str, int]
    right: tuple[str, int]
    applied: bool = False


@dataclass
class BossBullet:
    lane: float
    dist: float
    speed: float


@dataclass
class FloatingText:
    text: str
    x: float
    y: float
    color: tuple[int, int, int]
    timer: float = 0.95

    def update(self, dt: float) -> None:
        self.y -= 34 * dt
        self.timer -= dt


@dataclass
class Boss:
    dist: float
    target_dist: float
    hp: int
    max_hp: int
    attack_timer: float
    minion_timer: float
    phase: float = 0.0
    lane: float = 0.0


class Player:
    def __init__(self) -> None:
        self.lane = 0.0
        self.squad = 3
        self.fire_timer = 0.1
        self.invuln = 0.0
        self.hit_flash = 0.0

    @property
    def shot_count(self) -> int:
        return int(clamp(1 + self.squad // 3, 1, 8))

    @property
    def fire_interval(self) -> float:
        return clamp(0.42 - self.shot_count * 0.02, 0.24, 0.42)

    @property
    def formation_width(self) -> float:
        return 0.68 + min(1.15, self.squad * 0.045)

    def apply_operation(self, operation: tuple[str, int]) -> int:
        old_value = self.squad
        kind, value = operation
        if kind == "+":
            self.squad += value
        elif kind == "-":
            self.squad = max(1, self.squad - value)
        elif kind == "*":
            self.squad = min(MAX_SQUAD, self.squad * value)
        elif kind == "/":
            self.squad = max(1, math.ceil(self.squad / value))
        self.squad = int(clamp(self.squad, 1, MAX_SQUAD))
        return self.squad - old_value

    def take_damage(self, amount: int) -> None:
        self.squad -= amount
        self.invuln = 0.55
        self.hit_flash = 0.28


class NumberSquadRaid:
    def __init__(self) -> None:
        pygame.init()
        pygame.display.set_caption("Number Squad Highway")
        self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
        self.clock = pygame.time.Clock()
        self.rng = random.Random()

        self.font_small = pygame.font.Font(None, 28)
        self.font_medium = pygame.font.Font(None, 38)
        self.font_large = pygame.font.Font(None, 72)
        self.font_huge = pygame.font.Font(None, 98)

        self.state = "title"
        self.demo_scroll = 0.0
        self.reset_run()

    def reset_run(self) -> None:
        self.player = Player()
        self.bullets: list[Bullet] = []
        self.enemies: list[Enemy] = []
        self.gates: list[GatePair] = []
        self.boss_bullets: list[BossBullet] = []
        self.floating_texts: list[FloatingText] = []
        self.boss: Boss | None = None

        self.travel_distance = 0.0
        self.world_speed = 18.0
        self.enemy_timer = 0.75
        self.gate_timer = 3.6
        self.score = 0
        self.shake = 0.0
        self.flash = 0.0
        self.run_time = 0.0

    def start_game(self) -> None:
        self.reset_run()
        self.state = "play"

    def project_lane(self, lane: float, dist: float) -> tuple[float, float, float]:
        depth_ratio = 1.0 - clamp(dist / ROAD_DEPTH, 0.0, 1.0)
        depth_ratio = depth_ratio ** 1.05
        half_width = lerp(ROAD_TOP_HALF, ROAD_BOTTOM_HALF, depth_ratio)
        y = lerp(HORIZON_Y, ROAD_BOTTOM_Y, depth_ratio)
        x = ROAD_CENTER_X + (lane / LANE_SPAN) * half_width * 0.78
        scale = lerp(0.28, 1.22, depth_ratio)
        return x, y, scale

    def project_offset(self, offset_ratio: float, dist: float) -> tuple[float, float, float]:
        depth_ratio = 1.0 - clamp(dist / ROAD_DEPTH, 0.0, 1.0)
        depth_ratio = depth_ratio ** 1.05
        half_width = lerp(ROAD_TOP_HALF, ROAD_BOTTOM_HALF, depth_ratio)
        y = lerp(HORIZON_Y, ROAD_BOTTOM_Y, depth_ratio)
        x = ROAD_CENTER_X + offset_ratio * half_width
        scale = lerp(0.28, 1.22, depth_ratio)
        return x, y, scale

    def get_player_units(self) -> list[tuple[float, float, float, bool]]:
        base_x, base_y, scale = self.project_lane(self.player.lane, PLAYER_DIST)
        display_count = min(self.player.squad, 15)
        slots_per_row = 5
        units: list[tuple[float, float, float, bool]] = []

        for index in range(display_count - 1, -1, -1):
            row = index // slots_per_row
            index_in_row = index % slots_per_row
            remaining_in_row = min(slots_per_row, display_count - row * slots_per_row)
            offset_x = (index_in_row - (remaining_in_row - 1) / 2) * 22 * scale
            offset_y = row * 22 * scale
            unit_scale = scale * (0.95 - row * 0.05)
            units.append((base_x + offset_x, base_y + offset_y, unit_scale, index == 0))

        return units

    def ellipse_overlap(
        self,
        center_a: tuple[float, float],
        radius_a: tuple[float, float],
        center_b: tuple[float, float],
        radius_b: tuple[float, float],
    ) -> bool:
        combined_x = radius_a[0] + radius_b[0]
        combined_y = radius_a[1] + radius_b[1]
        if combined_x <= 0 or combined_y <= 0:
            return False
        dx = (center_a[0] - center_b[0]) / combined_x
        dy = (center_a[1] - center_b[1]) / combined_y
        return dx * dx + dy * dy <= 1.0

    def get_player_hitboxes(self) -> list[tuple[tuple[float, float], tuple[float, float]]]:
        hitboxes: list[tuple[tuple[float, float], tuple[float, float]]] = []
        for x, y, scale, _ in self.get_player_units():
            center = (x, y + 5.0 * scale)
            radii = (7.5 * scale, 14.0 * scale)
            hitboxes.append((center, radii))
        return hitboxes

    def get_enemy_hitbox(self, enemy: Enemy) -> tuple[tuple[float, float], tuple[float, float]]:
        x, y, scale = self.project_lane(enemy.lane, enemy.dist)
        scale_multiplier = 1.0 if enemy.kind == "runner" else 1.4
        if enemy.kind == "drifter":
            scale_multiplier = 1.15
        body_scale = scale * scale_multiplier
        center = (x, y + 1.5 * body_scale)
        radii = (9.0 * body_scale, 15.0 * body_scale)
        return center, radii

    def get_boss_bullet_hitbox(self, shot: BossBullet) -> tuple[tuple[float, float], tuple[float, float]]:
        x, y, scale = self.project_lane(shot.lane, shot.dist)
        center = (x, y + 11.0 * scale)
        radii = (4.5 * scale, 12.0 * scale)
        return center, radii

    def add_text(self, text: str, x: float, y: float, color: tuple[int, int, int]) -> None:
        self.floating_texts.append(FloatingText(text=text, x=x, y=y, color=color))

    def run(self) -> None:
        running = True
        while running:
            dt = min(0.033, self.clock.tick(FPS) / 1000)

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_ESCAPE:
                        running = False
                    if self.state in {"title", "gameover", "win"} and event.key in {
                        pygame.K_SPACE,
                        pygame.K_RETURN,
                        pygame.K_R,
                    }:
                        self.start_game()

            self.update(dt)
            self.draw()
            pygame.display.flip()

        pygame.quit()

    def update(self, dt: float) -> None:
        self.demo_scroll += dt * 9.0
        self.run_time += dt

        if self.state != "play":
            for text in self.floating_texts:
                text.update(dt)
            self.floating_texts = [text for text in self.floating_texts if text.timer > 0.0]
            return

        self.world_speed = 18.0 + min(8.0, self.travel_distance / 100.0)
        if self.boss:
            self.world_speed = 13.5

        self.travel_distance += self.world_speed * dt
        self.player.invuln = max(0.0, self.player.invuln - dt)
        self.player.hit_flash = max(0.0, self.player.hit_flash - dt)
        self.shake = max(0.0, self.shake - 28 * dt)
        self.flash = max(0.0, self.flash - 2.2 * dt)

        keys = pygame.key.get_pressed()
        move_left = keys[pygame.K_LEFT] or keys[pygame.K_a]
        move_right = keys[pygame.K_RIGHT] or keys[pygame.K_d]
        move_axis = float(move_right) - float(move_left)
        self.player.lane = clamp(self.player.lane + move_axis * 3.3 * dt, -LANE_SPAN, LANE_SPAN)

        self.player.fire_timer -= dt
        if self.player.fire_timer <= 0.0:
            self.fire_player_burst()
            self.player.fire_timer = self.player.fire_interval

        if not self.boss:
            self.enemy_timer -= dt
            self.gate_timer -= dt
            if self.enemy_timer <= 0.0:
                self.spawn_enemy_wave()
                difficulty = min(0.32, self.travel_distance / 1800.0)
                self.enemy_timer = self.rng.uniform(0.7, 1.15) - difficulty
            if self.gate_timer <= 0.0:
                self.spawn_gate_pair()
                self.gate_timer = self.rng.uniform(4.2, 5.6)
            if self.travel_distance >= BOSS_TRIGGER_DISTANCE:
                self.start_boss_fight()

        self.update_entities(dt)
        self.resolve_collisions()
        self.cleanup_entities()

        for text in self.floating_texts:
            text.update(dt)
        self.floating_texts = [text for text in self.floating_texts if text.timer > 0.0]

        if self.player.squad <= 0:
            self.state = "gameover"

    def update_entities(self, dt: float) -> None:
        for bullet in self.bullets:
            bullet.dist += bullet.speed * dt

        for enemy in self.enemies:
            enemy.dist -= (self.world_speed + enemy.speed) * dt
            if enemy.kind == "drifter":
                enemy.lane += math.sin(self.run_time * 3.2 + enemy.wobble) * 0.7 * dt
                enemy.lane = clamp(enemy.lane, -LANE_SPAN, LANE_SPAN)

        for gate in self.gates:
            gate.dist -= self.world_speed * dt

        for shot in self.boss_bullets:
            shot.dist -= shot.speed * dt

        if self.boss:
            self.boss.phase += dt
            if self.boss.dist > self.boss.target_dist:
                self.boss.dist -= 20.0 * dt
            self.boss.lane = math.sin(self.boss.phase * 1.1) * 1.15
            self.boss.attack_timer -= dt
            self.boss.minion_timer -= dt
            if self.boss.attack_timer <= 0.0 and self.boss.dist <= self.boss.target_dist + 1.0:
                self.spawn_boss_attack()
                pressure = 1.0 - (self.boss.hp / self.boss.max_hp)
                self.boss.attack_timer = 1.15 - pressure * 0.35
            if self.boss.minion_timer <= 0.0 and self.boss.dist <= self.boss.target_dist + 1.0:
                self.spawn_boss_minions()
                self.boss.minion_timer = 2.2

    def cleanup_entities(self) -> None:
        self.bullets = [bullet for bullet in self.bullets if bullet.dist < ROAD_DEPTH + 10]
        self.enemies = [enemy for enemy in self.enemies if enemy.hp > 0 and enemy.dist > -6]
        self.gates = [gate for gate in self.gates if gate.dist > -8]
        self.boss_bullets = [shot for shot in self.boss_bullets if shot.dist > -6]

    def fire_player_burst(self) -> None:
        shot_count = self.player.shot_count
        if shot_count == 1:
            lanes = [self.player.lane]
        else:
            spread = min(1.8, 0.34 * (shot_count - 1))
            lanes = [
                clamp(
                    self.player.lane + lerp(-spread, spread, index / (shot_count - 1)),
                    -LANE_SPAN,
                    LANE_SPAN,
                )
                for index in range(shot_count)
            ]

        for lane in lanes:
            self.bullets.append(Bullet(lane=lane, dist=PLAYER_DIST + 3.2, speed=82.0))

    def resolve_collisions(self) -> None:
        player_hitboxes = self.get_player_hitboxes()

        for gate in self.gates:
            if not gate.applied and gate.dist <= PLAYER_DIST + 2.0:
                operation = gate.left if self.player.lane <= 0 else gate.right
                delta = self.player.apply_operation(operation)
                x, y, _ = self.project_lane(self.player.lane, PLAYER_DIST + 4.0)
                color = GOOD_GREEN if delta >= 0 else BAD_RED
                sign = "+" if delta >= 0 else ""
                self.add_text(f"{format_operation(operation)}  {sign}{delta}", x, y - 46, color)
                self.score += max(0, delta) * 3
                gate.applied = True
                self.flash = 0.22

        for bullet in self.bullets[:]:
            hit = False
            for enemy in self.enemies:
                lane_window = 0.34 if enemy.kind != "brute" else 0.5
                if abs(bullet.dist - enemy.dist) < 3.2 and abs(bullet.lane - enemy.lane) < lane_window:
                    enemy.hp -= bullet.damage
                    hit = True
                    if enemy.hp <= 0:
                        self.score += enemy.reward
                        x, y, _ = self.project_lane(enemy.lane, max(enemy.dist, PLAYER_DIST + 6.0))
                        self.add_text("+10", x, y - 16, GOOD_GREEN)
                    break

            if not hit and self.boss:
                if abs(bullet.dist - self.boss.dist) < 7.0 and abs(bullet.lane - self.boss.lane) < 1.2:
                    self.boss.hp -= bullet.damage
                    hit = True
                    if self.boss.hp <= 0:
                        self.boss.hp = 0
                        x, y, _ = self.project_lane(self.boss.lane, self.boss.dist)
                        self.add_text("Boss Down!", x, y - 60, GOOD_GREEN)
                        self.score += 500
                        self.state = "win"

            if hit and bullet in self.bullets:
                self.bullets.remove(bullet)

        for enemy in self.enemies[:]:
            enemy_center, enemy_radii = self.get_enemy_hitbox(enemy)
            collided = any(
                self.ellipse_overlap(player_center, player_radii, enemy_center, enemy_radii)
                for player_center, player_radii in player_hitboxes
            )
            if collided:
                if self.player.invuln <= 0.0:
                    damage = 1 if enemy.kind == "runner" else 2
                    self.apply_player_damage(damage)
                enemy.hp = 0

        for shot in self.boss_bullets[:]:
            shot_center, shot_radii = self.get_boss_bullet_hitbox(shot)
            collided = any(
                self.ellipse_overlap(player_center, player_radii, shot_center, shot_radii)
                for player_center, player_radii in player_hitboxes
            )
            if collided:
                if self.player.invuln <= 0.0:
                    self.apply_player_damage(2)
                self.boss_bullets.remove(shot)

    def apply_player_damage(self, amount: int) -> None:
        self.player.take_damage(amount)
        x, y, _ = self.project_lane(self.player.lane, PLAYER_DIST)
        self.add_text(f"-{amount}", x, y - 58, BAD_RED)
        self.shake = 10.0
        self.flash = 0.18

    def spawn_enemy_wave(self) -> None:
        pattern = self.rng.choice(["line", "stream", "brute_wall", "drifters"])
        far_dist = ROAD_DEPTH - self.rng.uniform(4.0, 10.0)

        if pattern == "line":
            lanes = self.rng.sample([-2, -1, 0, 1, 2], k=self.rng.randint(2, 4))
            for lane in lanes:
                self.enemies.append(
                    Enemy(lane=lane, dist=far_dist, hp=1, speed=self.rng.uniform(3.0, 5.2), kind="runner", wobble=0.0, reward=10)
                )

        elif pattern == "stream":
            lane = self.rng.choice([-2, -1, 0, 1, 2])
            for index in range(5):
                self.enemies.append(
                    Enemy(
                        lane=lane,
                        dist=far_dist + index * 10,
                        hp=1,
                        speed=4.4,
                        kind="runner",
                        wobble=index * 0.5,
                        reward=10,
                    )
                )

        elif pattern == "brute_wall":
            lane_choices = self.rng.sample([-2, -1, 0, 1, 2], k=2)
            for lane in lane_choices:
                self.enemies.append(
                    Enemy(
                        lane=lane,
                        dist=far_dist,
                        hp=3,
                        speed=2.6,
                        kind="brute",
                        wobble=self.rng.random() * math.tau,
                        reward=24,
                    )
                )
            self.enemies.append(
                Enemy(
                    lane=self.rng.choice([-1, 0, 1]),
                    dist=far_dist + 8,
                    hp=1,
                    speed=4.0,
                    kind="runner",
                    wobble=0.0,
                    reward=10,
                )
            )

        elif pattern == "drifters":
            for lane in [-1.5, 0.0, 1.5]:
                self.enemies.append(
                    Enemy(
                        lane=lane,
                        dist=far_dist + self.rng.uniform(0.0, 8.0),
                        hp=2,
                        speed=3.6,
                        kind="drifter",
                        wobble=self.rng.random() * math.tau,
                        reward=16,
                    )
                )

    def gate_operation(self, helpful: bool) -> tuple[str, int]:
        safe_choices = [("+", 2), ("+", 3), ("+", 4), ("*", 2)]
        mixed_choices = [("+", 1), ("+", 2), ("-", 2), ("-", 3), ("/", 2), ("*", 2)]

        if helpful:
            if self.player.squad >= 16 and self.rng.random() < 0.35:
                return ("+", 5)
            return self.rng.choice(safe_choices)

        if self.player.squad <= 4:
            return self.rng.choice([("+", 1), ("+", 2), ("-", 1), ("*", 2)])

        choice = self.rng.choice(mixed_choices)
        if choice[0] == "-" and self.player.squad <= 3:
            return ("+", 2)
        return choice

    def spawn_gate_pair(self) -> None:
        left = self.gate_operation(helpful=self.rng.random() < 0.7)
        right = self.gate_operation(helpful=False)
        if self.rng.random() < 0.5:
            left, right = right, left
        self.gates.append(GatePair(dist=ROAD_DEPTH - 6.0, left=left, right=right))

    def start_boss_fight(self) -> None:
        self.boss = Boss(
            dist=ROAD_DEPTH - 5.0,
            target_dist=58.0,
            hp=220,
            max_hp=220,
            attack_timer=1.5,
            minion_timer=2.4,
        )
        self.gates.clear()
        x, y, _ = self.project_lane(0.0, 46.0)
        self.add_text("Boss Incoming!", x, y - 28, WARN_ORANGE)

    def spawn_boss_attack(self) -> None:
        if not self.boss:
            return
        pressure = 1.0 - (self.boss.hp / self.boss.max_hp)
        shot_count = 2 + int(pressure * 3)
        spread = 0.65 + pressure * 0.8
        for index in range(shot_count):
            if shot_count == 1:
                lane = self.player.lane
            else:
                lane = clamp(
                    self.player.lane + lerp(-spread, spread, index / (shot_count - 1)),
                    -LANE_SPAN,
                    LANE_SPAN,
                )
            self.boss_bullets.append(BossBullet(lane=lane, dist=self.boss.dist - 6.0, speed=28.0 + pressure * 4.0))

    def spawn_boss_minions(self) -> None:
        if not self.boss:
            return
        lanes = self.rng.sample([-2, -1, 0, 1, 2], k=2)
        for lane in lanes:
            self.enemies.append(
                Enemy(
                    lane=lane,
                    dist=self.boss.dist - self.rng.uniform(6.0, 12.0),
                    hp=2,
                    speed=3.8,
                    kind="runner",
                    wobble=0.0,
                    reward=14,
                )
            )

    def draw(self) -> None:
        scroll_value = self.travel_distance if self.state == "play" else self.demo_scroll
        self.draw_background(scroll_value)
        self.draw_road(scroll_value)
        self.draw_gates()
        self.draw_bullets()
        self.draw_enemies()
        self.draw_boss()
        self.draw_boss_bullets()
        self.draw_player()
        self.draw_floating_texts()
        self.draw_hud()
        self.draw_overlays()

    def draw_background(self, scroll_value: float) -> None:
        for y in range(HORIZON_Y):
            ratio = y / max(1, HORIZON_Y)
            color = blend(SKY_TOP, SKY_BOTTOM, ratio)
            pygame.draw.line(self.screen, color, (0, y), (WIDTH, y))

        sand_rect = pygame.Rect(0, HORIZON_Y, WIDTH, HEIGHT - HORIZON_Y)
        pygame.draw.rect(self.screen, SAND_FAR, sand_rect)

        pygame.draw.circle(self.screen, (255, 246, 195), (155, 104), 42)

        mesa_left = [(0, HORIZON_Y + 15), (42, 72), (172, 70), (194, HORIZON_Y + 14)]
        mesa_mid = [(160, HORIZON_Y + 28), (248, 88), (372, 84), (414, HORIZON_Y + 26)]
        mesa_right = [(WIDTH - 348, HORIZON_Y + 24), (WIDTH - 278, 76), (WIDTH - 118, 72), (WIDTH - 32, HORIZON_Y + 22)]
        for points, color in (
            (mesa_left, (201, 134, 94)),
            (mesa_mid, (187, 116, 86)),
            (mesa_right, (191, 120, 89)),
        ):
            pygame.draw.polygon(self.screen, color, points)

        for cloud_x, cloud_y, size in ((320, 58, 16), (468, 82, 12), (700, 52, 18)):
            self.draw_cloud(cloud_x, cloud_y, size)

        start_index = int(scroll_value // 26) - 2
        end_index = start_index + 10
        for world_index in range(start_index, end_index):
            dist = world_index * 26 - scroll_value
            if not 6 < dist < ROAD_DEPTH:
                continue
            side = -1 if math.sin(world_index * 1.73) > 0 else 1
            offset = 1.22 + 0.18 * abs(math.sin(world_index * 0.71))
            x, y, scale = self.project_offset(side * offset, dist)
            if world_index % 3 == 0:
                self.draw_rock(x, y, scale * 0.9)
            else:
                self.draw_cactus(x, y, scale * 0.95, side)

        fence_step = 14
        offset = scroll_value % fence_step
        for index in range(-1, int(ROAD_DEPTH / fence_step) + 4):
            dist = index * fence_step - offset
            if not 4 < dist < ROAD_DEPTH:
                continue
            for side in (-1, 1):
                x1, y1, scale1 = self.project_offset(side * 1.1, dist)
                x2, y2, scale2 = self.project_offset(side * 1.13, dist + 5)
                post_height = 18 * scale1
                pygame.draw.line(self.screen, (147, 100, 72), (x1, y1), (x1, y1 - post_height), max(1, int(2 * scale1)))
                pygame.draw.line(
                    self.screen,
                    (164, 116, 86),
                    (x1, y1 - post_height * 0.72),
                    (x2, y2 - post_height * 0.64),
                    max(1, int(2 * min(scale1, scale2))),
                )

    def draw_road(self, scroll_value: float) -> None:
        top_left = (ROAD_CENTER_X - ROAD_TOP_HALF, HORIZON_Y)
        top_right = (ROAD_CENTER_X + ROAD_TOP_HALF, HORIZON_Y)
        bottom_left = (ROAD_CENTER_X - ROAD_BOTTOM_HALF, ROAD_BOTTOM_Y)
        bottom_right = (ROAD_CENTER_X + ROAD_BOTTOM_HALF, ROAD_BOTTOM_Y)

        pygame.draw.polygon(
            self.screen,
            ROAD_EDGE,
            [
                (bottom_left[0] - 14, bottom_left[1]),
                (top_left[0] - 14, top_left[1]),
                (top_right[0] + 14, top_right[1]),
                (bottom_right[0] + 14, bottom_right[1]),
            ],
        )
        pygame.draw.polygon(self.screen, ROAD_COLOR, [bottom_left, top_left, top_right, bottom_right])

        dash_spacing = 12.0
        dash_length = 6.0
        lane_offsets = [(index / LANE_COUNT) * 2 - 1 for index in range(1, LANE_COUNT)]
        offset = scroll_value % dash_spacing
        for dist_index in range(-1, int(ROAD_DEPTH / dash_spacing) + 4):
            dist_start = dist_index * dash_spacing - offset
            dist_end = dist_start + dash_length
            if dist_end <= 0 or dist_start >= ROAD_DEPTH:
                continue
            for lane_offset in lane_offsets:
                x1, y1, scale1 = self.project_offset(lane_offset * 0.78, max(0.01, dist_start))
                x2, y2, scale2 = self.project_offset(lane_offset * 0.78, min(ROAD_DEPTH, dist_end))
                width = max(1, int(7 * min(scale1, scale2)))
                pygame.draw.line(self.screen, LANE_COLOR, (x1, y1), (x2, y2), width)

        side_glow = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
        pygame.draw.polygon(
            side_glow,
            (255, 255, 255, 20),
            [
                (bottom_left[0] + 40, bottom_left[1]),
                (top_left[0] + 14, top_left[1]),
                (top_right[0] - 14, top_right[1]),
                (bottom_right[0] - 40, bottom_right[1]),
            ],
        )
        self.screen.blit(side_glow, (0, 0))

    def draw_gates(self) -> None:
        overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
        for gate in sorted(self.gates, key=lambda item: item.dist, reverse=True):
            self.draw_gate_panel(overlay, gate.dist, -1.0, -0.02, gate.left, GATE_LEFT)
            self.draw_gate_panel(overlay, gate.dist, 0.02, 1.0, gate.right, GATE_RIGHT)
        self.screen.blit(overlay, (0, 0))
        for gate in sorted(self.gates, key=lambda item: item.dist, reverse=True):
            self.draw_gate_label(gate.dist, -1.0, -0.02, gate.left)
            self.draw_gate_label(gate.dist, 0.02, 1.0, gate.right)

    def draw_gate_panel(
        self,
        surface: pygame.Surface,
        dist: float,
        left_ratio: float,
        right_ratio: float,
        operation: tuple[str, int],
        color: tuple[int, int, int, int],
    ) -> None:
        front_left = self.project_offset(left_ratio, dist)
        front_right = self.project_offset(right_ratio, dist)
        back_left = self.project_offset(left_ratio, dist + 6.8)
        back_right = self.project_offset(right_ratio, dist + 6.8)
        points = [
            (front_left[0], front_left[1]),
            (back_left[0], back_left[1]),
            (back_right[0], back_right[1]),
            (front_right[0], front_right[1]),
        ]
        pygame.draw.polygon(surface, color, points)
        pygame.draw.polygon(surface, (255, 255, 255, 60), points, 2)

    def draw_gate_label(
        self,
        dist: float,
        left_ratio: float,
        right_ratio: float,
        operation: tuple[str, int],
    ) -> None:
        front_left = self.project_offset(left_ratio, dist)
        front_right = self.project_offset(right_ratio, dist)
        back_left = self.project_offset(left_ratio, dist + 6.8)
        back_right = self.project_offset(right_ratio, dist + 6.8)
        points = [
            (front_left[0], front_left[1]),
            (back_left[0], back_left[1]),
            (back_right[0], back_right[1]),
            (front_right[0], front_right[1]),
        ]
        label_x = sum(point[0] for point in points) / 4
        label_y = sum(point[1] for point in points) / 4
        label = format_operation(operation)
        label_surface = self.font_large.render(label, True, WHITE)
        label_surface = pygame.transform.smoothscale(
            label_surface,
            (
                int(label_surface.get_width() * clamp(front_left[2] * 0.92, 0.5, 1.5)),
                int(label_surface.get_height() * clamp(front_left[2] * 0.92, 0.5, 1.5)),
            ),
        )
        rect = label_surface.get_rect(center=(label_x, label_y))
        outline = self.font_large.render(label, True, TEXT_DARK)
        outline = pygame.transform.smoothscale(outline, label_surface.get_size())
        for dx, dy in ((-2, 0), (2, 0), (0, -2), (0, 2)):
            self.screen.blit(outline, outline.get_rect(center=(label_x + dx, label_y + dy)))
        self.screen.blit(label_surface, rect)

    def draw_bullets(self) -> None:
        for bullet in sorted(self.bullets, key=lambda item: item.dist, reverse=True):
            x, y, scale = self.project_lane(bullet.lane, bullet.dist)
            length = 18 * scale
            width = max(2, int(4 * scale))
            pygame.draw.line(self.screen, BULLET_GOLD, (x, y), (x, y - length), width)
            pygame.draw.circle(self.screen, (255, 238, 163), (int(x), int(y - length)), max(2, int(5 * scale)))

    def draw_enemy_body(self, x: float, y: float, scale: float, kind: str) -> None:
        scale_multiplier = 1.0 if kind == "runner" else 1.4
        if kind == "drifter":
            scale_multiplier = 1.15
        body_scale = scale * scale_multiplier

        shadow_rect = pygame.Rect(0, 0, 36 * body_scale, 12 * body_scale)
        shadow_rect.center = (x, y + 8 * body_scale)
        pygame.draw.ellipse(self.screen, (0, 0, 0, 70), shadow_rect)

        leg_y = y + 16 * body_scale
        pygame.draw.line(self.screen, ENEMY_DARK, (x - 7 * body_scale, y + 6 * body_scale), (x - 9 * body_scale, leg_y), max(2, int(3 * body_scale)))
        pygame.draw.line(self.screen, ENEMY_DARK, (x + 7 * body_scale, y + 6 * body_scale), (x + 9 * body_scale, leg_y), max(2, int(3 * body_scale)))
        pygame.draw.line(self.screen, ENEMY_DARK, (x, y - 4 * body_scale), (x, y + 8 * body_scale), max(3, int(6 * body_scale)))
        pygame.draw.line(self.screen, ENEMY_DARK, (x - 9 * body_scale, y + 2 * body_scale), (x + 9 * body_scale, y + 2 * body_scale), max(2, int(3 * body_scale)))
        pygame.draw.circle(self.screen, ENEMY_RED, (int(x), int(y - 12 * body_scale)), max(5, int(9 * body_scale)))

    def draw_enemies(self) -> None:
        for enemy in sorted(self.enemies, key=lambda item: item.dist, reverse=True):
            x, y, scale = self.project_lane(enemy.lane, enemy.dist)
            self.draw_enemy_body(x, y, scale, enemy.kind)
            if enemy.hp > 1:
                hp_label = self.font_small.render(str(enemy.hp), True, WHITE)
                self.screen.blit(hp_label, hp_label.get_rect(center=(x, y - 24 * scale)))

    def draw_boss(self) -> None:
        if not self.boss:
            return
        x, y, scale = self.project_lane(self.boss.lane, self.boss.dist)
        width = 200 * scale
        height = 128 * scale

        shadow = pygame.Rect(0, 0, width * 0.9, height * 0.24)
        shadow.center = (x, y + height * 0.3)
        pygame.draw.ellipse(self.screen, (0, 0, 0, 80), shadow)

        body = pygame.Rect(0, 0, width, height * 0.62)
        body.center = (x, y)
        turret = pygame.Rect(0, 0, width * 0.56, height * 0.36)
        turret.center = (x, y - height * 0.16)

        pygame.draw.rect(self.screen, BOSS_DARK, body, border_radius=int(18 * scale))
        pygame.draw.rect(self.screen, BOSS_PURPLE, body.inflate(-width * 0.12, -height * 0.18), border_radius=int(16 * scale))
        pygame.draw.rect(self.screen, BOSS_DARK, turret, border_radius=int(16 * scale))
        pygame.draw.rect(self.screen, (220, 186, 255), turret.inflate(-width * 0.16, -height * 0.18), border_radius=int(12 * scale))

        cannon_y = y - height * 0.1
        cannon_width = 14 * scale
        cannon_length = 72 * scale
        pygame.draw.rect(
            self.screen,
            BOSS_DARK,
            pygame.Rect(x - cannon_width / 2, cannon_y - cannon_length, cannon_width, cannon_length),
            border_radius=max(4, int(5 * scale)),
        )
        eye_rect = pygame.Rect(0, 0, width * 0.16, height * 0.07)
        eye_rect.center = (x, y - height * 0.16)
        pygame.draw.rect(self.screen, (255, 111, 197), eye_rect, border_radius=max(4, int(4 * scale)))

        wheel_radius = max(5, int(14 * scale))
        for side in (-1, 1):
            wheel_center = (int(x + side * width * 0.34), int(y + height * 0.12))
            pygame.draw.circle(self.screen, BOSS_DARK, wheel_center, wheel_radius)
            pygame.draw.circle(self.screen, (255, 206, 120), wheel_center, max(2, wheel_radius // 3))

    def draw_boss_bullets(self) -> None:
        for shot in sorted(self.boss_bullets, key=lambda item: item.dist, reverse=True):
            x, y, scale = self.project_lane(shot.lane, shot.dist)
            pygame.draw.line(self.screen, (255, 111, 197), (x, y), (x, y + 22 * scale), max(3, int(5 * scale)))
            pygame.draw.circle(self.screen, (255, 197, 234), (int(x), int(y + 22 * scale)), max(3, int(5 * scale)))

    def draw_player_unit(self, x: float, y: float, scale: float, leader: bool) -> None:
        helmet_radius = max(6, int(10 * scale))
        ring_radius = max(8, int(16 * scale))
        body_y = y + 14 * scale
        body_color = (228, 240, 255) if self.player.hit_flash > 0 else (222, 234, 255)
        helmet_color = (129, 184, 255) if leader else PLAYER_BLUE

        ring_rect = pygame.Rect(0, 0, ring_radius * 2.1, ring_radius * 0.9)
        ring_rect.center = (x, y + 22 * scale)
        pygame.draw.ellipse(self.screen, PLAYER_RING, ring_rect, max(2, int(3 * scale)))

        pygame.draw.line(self.screen, body_color, (x, y - 2 * scale), (x, body_y), max(2, int(4 * scale)))
        pygame.draw.line(self.screen, body_color, (x - 7 * scale, y + 6 * scale), (x + 7 * scale, y + 3 * scale), max(2, int(3 * scale)))
        pygame.draw.line(self.screen, body_color, (x - 4 * scale, body_y), (x - 8 * scale, body_y + 8 * scale), max(2, int(3 * scale)))
        pygame.draw.line(self.screen, body_color, (x + 4 * scale, body_y), (x + 8 * scale, body_y + 8 * scale), max(2, int(3 * scale)))
        pygame.draw.circle(self.screen, helmet_color, (int(x), int(y - 10 * scale)), helmet_radius)
        pygame.draw.line(self.screen, (19, 34, 64), (x + 3 * scale, y + 4 * scale), (x + 14 * scale, y + 1 * scale), max(2, int(3 * scale)))

    def draw_player(self) -> None:
        units = self.get_player_units()
        base_x, base_y, scale = self.project_lane(self.player.lane, PLAYER_DIST)

        for unit_x, unit_y, unit_scale, leader in units:
            self.draw_player_unit(unit_x, unit_y, unit_scale, leader)

        if self.player.squad > len(units):
            bubble_text = self.font_medium.render(f"x{self.player.squad}", True, WHITE)
            bubble_rect = bubble_text.get_rect(center=(base_x, base_y - 54 * scale))
            bubble_back = bubble_rect.inflate(20, 12)
            pygame.draw.rect(self.screen, (40, 60, 98), bubble_back, border_radius=16)
            pygame.draw.rect(self.screen, (126, 188, 255), bubble_back, 2, border_radius=16)
            self.screen.blit(bubble_text, bubble_rect)

    def draw_floating_texts(self) -> None:
        for text in self.floating_texts:
            alpha = int(clamp(text.timer / 0.95, 0.0, 1.0) * 255)
            base = self.font_medium.render(text.text, True, text.color)
            outline = self.font_medium.render(text.text, True, TEXT_DARK)
            base.set_alpha(alpha)
            outline.set_alpha(alpha)
            for dx, dy in ((-2, 0), (2, 0), (0, -2), (0, 2)):
                self.screen.blit(outline, outline.get_rect(center=(text.x + dx, text.y + dy)))
            self.screen.blit(base, base.get_rect(center=(text.x, text.y)))

    def draw_hud(self) -> None:
        hud_panel = pygame.Rect(18, 14, 280, 110)
        pygame.draw.rect(self.screen, (18, 28, 50, 185), hud_panel, border_radius=18)
        pygame.draw.rect(self.screen, (120, 177, 255), hud_panel, 2, border_radius=18)

        stats = [
            f"Squad  {self.player.squad}",
            f"Burst  {self.player.shot_count}",
            f"Score  {self.score}",
        ]
        for index, line in enumerate(stats):
            label = self.font_medium.render(line, True, WHITE)
            self.screen.blit(label, (34, 28 + index * 28))

        progress_label = "Boss Fight" if self.boss else "Boss Inbound"
        bar_x, bar_y, bar_w, bar_h = 320, 22, 360, 26
        pygame.draw.rect(self.screen, (32, 34, 44), (bar_x, bar_y, bar_w, bar_h), border_radius=14)
        if self.boss:
            ratio = self.boss.hp / self.boss.max_hp
            fill_color = (255, 103, 191)
        else:
            ratio = clamp(self.travel_distance / BOSS_TRIGGER_DISTANCE, 0.0, 1.0)
            fill_color = (255, 108, 108)
        pygame.draw.rect(
            self.screen,
            fill_color,
            (bar_x, bar_y, bar_w * ratio, bar_h),
            border_radius=14,
        )
        pygame.draw.rect(self.screen, WHITE, (bar_x, bar_y, bar_w, bar_h), 2, border_radius=14)

        label = self.font_medium.render(progress_label, True, WHITE)
        self.screen.blit(label, label.get_rect(center=(bar_x + bar_w / 2, bar_y + bar_h / 2)))

        if self.state == "play" and self.travel_distance < 90:
            tip = self.font_small.render("Move with A / D or Left / Right. Choose a gate side, then survive to the boss.", True, TEXT_DARK)
            self.screen.blit(tip, (26, HEIGHT - 34))

    def draw_overlays(self) -> None:
        if self.flash > 0.0:
            overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
            overlay.fill((255, 255, 255, int(self.flash * 110)))
            self.screen.blit(overlay, (0, 0))

        if self.state == "title":
            overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
            overlay.fill((8, 12, 24, 96))
            self.screen.blit(overlay, (0, 0))

            title = self.font_huge.render("NUMBER SQUAD", True, WHITE)
            subtitle = self.font_large.render("Highway Raid", True, (255, 220, 147))
            desc = self.font_medium.render("Grow your math squad, flood the road with bullets, beat the boss.", True, WHITE)
            prompt = self.font_medium.render("Press Space to start", True, GOOD_GREEN)

            self.screen.blit(title, title.get_rect(center=(WIDTH / 2, 150)))
            self.screen.blit(subtitle, subtitle.get_rect(center=(WIDTH / 2, 214)))
            self.screen.blit(desc, desc.get_rect(center=(WIDTH / 2, 282)))
            self.screen.blit(prompt, prompt.get_rect(center=(WIDTH / 2, 344)))

            tips = [
                "Positive gates make your squad larger and your bullet burst wider.",
                "Enemies that reach your lane will shrink the squad.",
                "The boss spawns after the road sprint phase.",
            ]
            for index, line in enumerate(tips):
                tip = self.font_small.render(line, True, WHITE)
                self.screen.blit(tip, tip.get_rect(center=(WIDTH / 2, 404 + index * 26)))

        elif self.state in {"gameover", "win"}:
            overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
            overlay.fill((10, 12, 20, 154))
            self.screen.blit(overlay, (0, 0))

            headline = "Stage Clear" if self.state == "win" else "Squad Wiped"
            color = GOOD_GREEN if self.state == "win" else BAD_RED
            title = self.font_huge.render(headline, True, color)
            score = self.font_large.render(f"Final Score  {self.score}", True, WHITE)
            prompt = self.font_medium.render("Press Space or R to play again", True, WHITE)
            self.screen.blit(title, title.get_rect(center=(WIDTH / 2, 220)))
            self.screen.blit(score, score.get_rect(center=(WIDTH / 2, 300)))
            self.screen.blit(prompt, prompt.get_rect(center=(WIDTH / 2, 360)))

    def draw_cloud(self, x: float, y: float, size: int) -> None:
        for dx, dy, radius in ((0, 0, size), (size, -5, int(size * 0.8)), (-size, -2, int(size * 0.7))):
            pygame.draw.circle(self.screen, (255, 255, 255), (x + dx, y + dy), radius)

    def draw_cactus(self, x: float, y: float, scale: float, side: int) -> None:
        trunk_width = max(4, int(10 * scale))
        trunk_height = max(14, int(34 * scale))
        arm_width = max(3, int(7 * scale))
        arm_height = max(8, int(18 * scale))

        trunk = pygame.Rect(0, 0, trunk_width, trunk_height)
        trunk.midbottom = (x, y)
        pygame.draw.rect(self.screen, (50, 126, 78), trunk, border_radius=max(3, int(4 * scale)))

        arm = pygame.Rect(0, 0, arm_width, arm_height)
        arm.midbottom = (x - side * trunk_width, y - trunk_height * 0.35)
        pygame.draw.rect(self.screen, (60, 142, 88), arm, border_radius=max(2, int(4 * scale)))
        pygame.draw.line(
            self.screen,
            (60, 142, 88),
            (arm.centerx, arm.top + 4 * scale),
            (arm.centerx + side * 10 * scale, arm.top + 4 * scale),
            max(2, int(3 * scale)),
        )

    def draw_rock(self, x: float, y: float, scale: float) -> None:
        width = 18 * scale
        height = 12 * scale
        points = [
            (x - width, y),
            (x - width * 0.5, y - height),
            (x + width * 0.3, y - height * 0.8),
            (x + width, y),
            (x + width * 0.4, y + height * 0.2),
            (x - width * 0.8, y + height * 0.1),
        ]
        pygame.draw.polygon(self.screen, (210, 177, 148), points)


if __name__ == "__main__":
    NumberSquadRaid().run()

```