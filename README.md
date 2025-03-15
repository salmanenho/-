# -from kivy.app import App
from kivy.uix.widget import Widget
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.popup import Popup
from kivy.clock import Clock
from kivy.core.window import Window
from kivy.graphics import Color, Rectangle
from kivy.uix.gridlayout import GridLayout
import random

# إعدادات اللعبة
WIDTH = 800
HEIGHT = 600
NEEDLE_SPEED = 5
VIRUS_COUNT = 10
VITAMIN_COUNT = 5

# فئة الإبرة
class Needle(Widget):
    def __init__(self, color, name, **kwargs):
        super().__init__(**kwargs)
        self.size = (30, 5)
        self.color = color
        self.name = name
        self.score = 0
        self.direction = [random.choice([-1, 1]), random.choice([-1, 1])]
        with self.canvas:
            Color(*self.color)
            self.rect = Rectangle(pos=self.pos, size=self.size)

    def update(self):
        self.x += self.direction[0] * NEEDLE_SPEED
        self.y += self.direction[1] * NEEDLE_SPEED
        # تجنب الخروج من الشاشة
        if self.x < 0 or self.x > WIDTH - self.width:
            self.direction[0] *= -1
        if self.y < 0 or self.y > HEIGHT - self.height:
            self.direction[1] *= -1
        self.rect.pos = self.pos

# فئة الفيروس
class Virus(Widget):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.size = (20, 20)
        with self.canvas:
            Color(1, 0, 0)  # أحمر
            self.rect = Rectangle(pos=self.pos, size=self.size)

# فئة الفيتامين
class Vitamin(Widget):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.size = (15, 15)
        with self.canvas:
            Color(0, 1, 0)  # أخضر
            self.rect = Rectangle(pos=self.pos, size=self.size)

# الشاشة الرئيسية للعبة
class GameScreen(Widget):
    def __init__(self, players, **kwargs):
        super().__init__(**kwargs)
        self.players = players
        self.needles = []
        self.viruses = []
        self.vitamins = []
        self.setup_game()

    def setup_game(self):
        # إنشاء الإبر
        for player in self.players:
            needle = Needle(player["color"], player["name"])
            needle.pos = (random.randint(0, WIDTH - 30), random.randint(0, HEIGHT - 5))
            self.add_widget(needle)
            self.needles.append(needle)

        # إنشاء الفيروسات
        for _ in range(VIRUS_COUNT):
            virus = Virus()
            virus.pos = (random.randint(0, WIDTH - 20), random.randint(0, HEIGHT - 20))
            self.add_widget(virus)
            self.viruses.append(virus)

        # إنشاء الفيتامينات
        for _ in range(VITAMIN_COUNT):
            vitamin = Vitamin()
            vitamin.pos = (random.randint(0, WIDTH - 15), random.randint(0, HEIGHT - 15))
            self.add_widget(vitamin)
            self.vitamins.append(vitamin)

        # بدء التحديث التلقائي
        Clock.schedule_interval(self.update, 1/30)

    def update(self, dt):
        for needle in self.needles:
            needle.update()
            # الكشف عن التصادم مع الفيتامينات
            for vitamin in self.vitamins[:]:
                if needle.collide_widget(vitamin):
                    self.vitamins.remove(vitamin)
                    self.remove_widget(vitamin)
                    needle.score += 10
                    # إضافة فيتامين جديد
                    new_vitamin = Vitamin()
                    new_vitamin.pos = (random.randint(0, WIDTH - 15), random.randint(0, HEIGHT - 15))
                    self.add_widget(new_vitamin)
                    self.vitamins.append(new_vitamin)

            # الكشف عن التصادم مع الفيروسات
            for virus in self.viruses:
                if needle.collide_widget(virus):
                    self.show_game_over(needle)
                    self.needles.remove(needle)
                    self.remove_widget(needle)
                    break

    def show_game_over(self, needle):
        popup = Popup(title="Game Over", size_hint=(0.6, 0.4))
        content = BoxLayout(orientation="vertical")
        content.add_widget(Label(text=f"{needle.name} خسر! النقاط النهائية: {needle.score}"))
        popup.content = content
        popup.open()

# واجهة إدخال أسماء اللاعبين
class PlayerInputScreen(BoxLayout):
    def __init__(self, start_game_callback, **kwargs):
        super().__init__(**kwargs)
        self.orientation = "vertical"
        self.start_game_callback = start_game_callback
        self.player_inputs = []

        self.add_widget(Label(text="أدخل عدد اللاعبين:"))
        self.num_players_input = TextInput(multiline=False)
        self.add_widget(self.num_players_input)

        self.add_widget(Button(text="ابدأ", on_press=self.start_game))

    def start_game(self, instance):
        num_players = int(self.num_players_input.text)
        players = []
        for i in range(num_players):
            name = input(f"أدخل اسم اللاعب {i+1}: ")
            color = (random.random(), random.random(), random.random())  # لون عشوائي
            players.append({"name": name, "color": color})
        self.start_game_callback(players)

# تطبيق Kivy
class NeedleGameApp(App):
    def build(self):
        self.title = "لعبة الإبرة"
        self.screen_manager = BoxLayout(orientation="vertical")
        self.player_input_screen = PlayerInputScreen(self.start_game)
        self.screen_manager.add_widget(self.player_input_screen)
        return self.screen_manager

    def start_game(self, players):
        self.screen_manager.clear_widgets()
        self.game_screen = GameScreen(players)
        self.screen_manager.add_widget(self.game_screen)

# تشغيل التطبيق
if __name__ == "__main__":
    Window.size = (WIDTH, HEIGHT)
    NeedleGameApp().run()
