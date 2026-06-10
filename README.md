full gaming code
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.clock import Clock
import json
import os

SAVE_FILE = "simulator_save.json"


class Simulator(BoxLayout):

    def __init__(self, **kwargs):
        super().__init__(orientation="vertical", **kwargs)

        self.level = 1
        self.xp = 0
        self.coins = 0
        self.power = 1
        self.workers = 0

        self.load_game()

        self.stats = Label(font_size=20)
        self.add_widget(self.stats)

        earn_btn = Button(text="Earn Coins", font_size=24)
        earn_btn.bind(on_press=self.earn)
        self.add_widget(earn_btn)

        upgrade_btn = Button(text="Upgrade Power")
        upgrade_btn.bind(on_press=self.upgrade)
        self.add_widget(upgrade_btn)

        worker_btn = Button(text="Hire Worker")
        worker_btn.bind(on_press=self.hire_worker)
        self.add_widget(worker_btn)

        save_btn = Button(text="Save Game")
        save_btn.bind(on_press=self.save_game)
        self.add_widget(save_btn)

        Clock.schedule_interval(self.passive_income, 1)
        self.update_ui()

    def xp_needed(self):
        return self.level * 100

    def update_ui(self):
        self.stats.text = (
            f"Level: {self.level}\n"
            f"XP: {self.xp}/{self.xp_needed()}\n"
            f"Coins: {self.coins}\n"
            f"Power: {self.power}\n"
            f"Workers: {self.workers}"
        )

    def earn(self, instance):
        gain = self.power * 5
        self.coins += gain
        self.xp += 5

        while self.xp >= self.xp_needed():
            self.xp -= self.xp_needed()
            self.level += 1
            self.power += 1

        self.update_ui()

    def upgrade(self, instance):
        cost = self.power * 100

        if self.coins >= cost:
            self.coins -= cost
            self.power += 1

        self.update_ui()

    def hire_worker(self, instance):
        cost = (self.workers + 1) * 250

        if self.coins >= cost:
            self.coins -= cost
            self.workers += 1

        self.update_ui()

    def passive_income(self, dt):
        self.coins += self.workers
        self.update_ui()

    def save_game(self, instance=None):
        data = {
            "level": self.level,
            "xp": self.xp,
            "coins": self.coins,
            "power": self.power,
            "workers": self.workers
        }

        with open(SAVE_FILE, "w") as f:
            json.dump(data, f)

    def load_game(self):
        if os.path.exists(SAVE_FILE):
            try:
                with open(SAVE_FILE, "r") as f:
                    data = json.load(f)

                self.level = data.get("level", 1)
                self.xp = data.get("xp", 0)
                self.coins = data.get("coins", 0)
                self.power = data.get("power", 1)
                self.workers = data.get("workers", 0)

            except:
                pass


class SimulatorApp(App):
    def build(self):
        return Simulator()


if __name__ == "__main__":
    SimulatorApp().run()
