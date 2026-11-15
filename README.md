# web#!/usr/bin/env python3
"""
Mission Control: NASA Space Apps Kırklareli  (terminal / Python version)

Python port of the interactive HTML story game. Same story, same choices,
same win/fail rules. Run:  python3 mission_control.py
Options:  --fast   skip the typing animation
"""
import sys
import time

FAST = "--fast" in sys.argv or not sys.stdout.isatty()

WHO = {
    "g": "Guide Instructor",
    "n": "Nebula",
    "h": "Houston, via Nebula",
}


class MissionFailed(Exception):
    """Raised when a choice makes the mission impossible to continue."""


# ---------------------------------------------------------------- helpers
def typewriter(text, delay=0.012):
    if FAST:
        print(text)
        return
    for ch in text:
        sys.stdout.write(ch)
        sys.stdout.flush()
        time.sleep(delay)
    print()


def banner(text):
    line = "=" * (len(text) + 8)
    print(f"\n{line}\n==  {text}  ==\n{line}")
    if not FAST:
        time.sleep(1.2)


def scene(name):
    print(f"\n--- [ scene: {name} ] ---")


def say(text, who="n"):
    print(f"\n[{WHO[who]}]")
    typewriter(text)
    input("   Next ▶ (Enter) ")


def ask(question, options, who="n"):
    """options: list of (label, value, hint). Returns the chosen value."""
    print(f"\n[{WHO[who]}]")
    typewriter(question)
    for i, (label, _value, hint) in enumerate(options, 1):
        extra = f"  ({hint})" if hint else ""
        print(f"   {i}) {label}{extra}")
    while True:
        raw = input("   Your choice: ").strip()
        if raw.isdigit() and 1 <= int(raw) <= len(options):
            return options[int(raw) - 1][1]
        print(f"   Please enter a number from 1 to {len(options)}.")


def fail(message):
    raise MissionFailed(message)


def name_step():
    print(f"\n[{WHO['g']}]")
    typewriter("Hello, future astronaut! What's your name?")
    print("   (suggestions: Nova, Orion, Luna)")
    name = input("   Your name: ").strip()[:18]
    return name or "Commander"


def fill_tubes(kind):
    label = "leaf" if kind == "leaf" else "soil"
    print(f"\n[{WHO['n']}]")
    typewriter(f"Tap each tube to fill it! ({label} samples)")
    tubes = ["🧪"] * 5
    for i in range(5):
        print("   " + " ".join(tubes))
        input(f"   Press Enter to fill tube {i + 1}/5 ")
        tubes[i] = "✅"
    print("   " + " ".join(tubes))


# ---------------------------------------------------------------- story
def orion_tour():
    scene("Orion capsule tour")
    tour = [
        "Today I will introduce you to our capsule, Orion. Orion was created with inspiration from the Apollo-era capsules. Now, let’s take a closer look at the capsule that will take us to Mars!",
        "Orion is NASA’s deep-space crew capsule. It is built to carry astronauts farther than humans have travelled in over 50 years, far beyond low Earth orbit.",
        "It has two main parts. The Crew Module is where we live and work. Below it sits the European Service Module, built in Europe, which provides power, water, air, and propulsion.",
        "Remember the 12 solar panels I mentioned? They sit on the Service Module: four wings, each with three panels, reaching out like a big X. They turn sunlight into the power for everything onboard.",
        "Orion looks like an Apollo capsule, but it is bigger and far more advanced. It can carry up to four astronauts, and it has about 50% more room inside than Apollo had.",
        "At the very bottom is the heat shield, about 5 meters wide. When Orion returns to Earth at roughly 40,000 km/h, it faces temperatures near 2,700 °C, almost half as hot as the surface of the Sun!",
        "On top of the capsule sits the Launch Abort System. If anything goes wrong during liftoff, it can pull the crew to safety in a split second.",
        "Orion has already proven itself: in 2022 the uncrewed Artemis I mission flew it around the Moon and brought it safely home. Now it is our turn to take it even farther!",
    ]
    for line in tour:
        say(line, "g")


def play():
    v = {}
    scene("Vehicle Assembly Building")
    name = name_step()
    v["name"] = name
    say(f"Welcome, Commander {name}! As your Guide Spacecraft Instructor, I will introduce you to the space capsule and help you with your pre-mission choices.", "g")
    say("Please do not forget: every choice you make will have consequences later in the mission.", "g")
    orion_tour()

    # ---- DAY 1
    banner("DAY 1")
    say("Welcome back, Commander. Our first choice today is the UV shield.", "g")
    say("Ultraviolet (UV) rays come from stars such as the Sun. We cannot see them with our eyes. Our atmosphere protects us by blocking a large part of these rays.", "g")
    v["uv"] = ask("What kind of UV protection would you like to take with you?",
                  [("Low", "low", ""), ("Medium", "medium", ""),
                   ("High", "high", "heavy, but safest")], "g")
    say("The Orion capsule has 12 solar panels. You can also take 4 extra panels along. If you decide not to, we will include a surprise item instead.", "g")
    say("Solar panels are special devices that turn sunlight into electrical energy. They keep the technology on our space base running.", "g")
    v["panel"] = ask("Would you like to take the extra solar panels?",
                     [("Yes, take them! ☀️", "yes", ""),
                      ("No, surprise me! 🎁", "no", "")], "g")

    # ---- DAY 2
    banner("DAY 2")
    say("Garden construction kits will make your job much easier. They come in three sizes: small, medium, and large. Each gives you different benefits, but do not forget the weight limit!", "g")
    gardens = [("Large 🌳", "large", "needs low or medium UV"),
               ("Medium 🌿", "medium", ""), ("Small 🌱", "small", "")]
    if v["uv"] == "high":
        say("To stay within the weight limit, you cannot choose a large garden together with high UV protection.", "g")
        gardens = gardens[1:]
    v["garden"] = ask("Choose the size of your garden-building kit!", gardens, "g")

    # ---- DAY 3
    banner("DAY 3")
    say("On Mars there are sometimes earthquakes and enormous dust storms. These storms can cover the sky and reduce the sunlight that reaches the surface. Sometimes they even wrap the whole planet.", "g")
    say("A repair kit can save you in difficult situations. It will be one of your biggest friends during tremors and dust storms.", "g")
    v["kit"] = ask("Do you want a repair kit?",
                   [("Yes 🧰", "yes", ""), ("No", "no", "")], "g")
    say("Test tubes are the most important tool for your purpose. There are two types: a biological set for mixing larger amounts of liquid, and a microtube set for much smaller amounts. Both help scientists run experiments. Choose carefully!", "g")
    v["tubes"] = ask("Which test tube set do you choose?",
                     [("Cold biological set ❄️", "bio", ""),
                      ("Normal microtube set", "micro", "")], "g")

    # ---- DAY 4
    banner("DAY 4")
    say("Food storage means preserving, ahead of time, the food that astronauts need. There is no market on Mars, so you need enough food to last a long time.", "g")
    v["stock"] = ask("Your last decision before departure: how much food do you take?",
                     [("2.5 weeks 🥫", "2.5", ""), ("6 weeks 🥫🥫", "6", "")], "g")
    say("I hope you chose well, kid! The trip will take 9 months, but you will still grow food there. A piece of advice: sometimes you need more than you think.", "g")

    scene("Launch")
    say("Now that you have made your last choice, it is time for liftoff. The journey will take nine months. Houston will not call you during that time. See you soon... 🚀", "g")
    scene("Orbit")
    banner("9 MONTHS LATER")
    say(f"Congratulations, {name}! You have completed your nine-month space journey. You are now on Mars! I am Nebula, your assistant. I will relay messages from Houston and help you with your choices.")

    # ---- MARS
    scene("Mars")
    say("You chose extra solar panels, so I have installed them for you."
        if v["panel"] == "yes" else
        "Your surprise item is a spare power cell, so I have stored it safely.")
    say("Now it is time to set up your garden!")
    say(f"Your {v['garden']} garden is growing nicely. Next, choose how to stabilize it with the life support unit.")
    v["power"] = ask("How should the life support unit share its energy?",
                     [("A · Equal energy", "A", "easy to do"),
                      ("B · 45% temperature, 30% water, 25% oxygen", "B", "a good option"),
                      ("C · Adjust to time and environment", "C", "very good, but may need manual settings")])
    say("Since you have set up the garden and the life-support unit, that is enough for today. See you tomorrow!")

    # ---- NIGHT ONE
    scene("Houston HQ")
    banner("NIGHT ONE")
    say("Seismic anomalies were detected tonight near Houston. This could indicate an earthquake.", "h")
    v["quake"] = ask("Do you want to take precautions against the earthquake?",
                     [("Continue my daily tasks", "no", ""),
                      ("Secure the capsule and the garden", "yes", "")], "h")
    scene("Mars")
    say("Houston has detected a severe solar storm in our region.", "h")
    if v["uv"] == "low":
        fail("Because you have low UV protection, you cannot withstand any more radiation. You must cut the mission short.")
    say("Fortunately, you have the right level of UV protection. The mission will continue.")
    say("A magnitude 4.2 earthquake has just occurred. It seems small, but on Mars the planet’s internal structure makes quakes ring for about 6 hours. A seismic anomaly is any unusual ground shaking. Scientists study it to understand what is happening inside the planet.")
    if v["quake"] == "yes":
        say("Thanks to your precautions, there was no damage to the garden or the systems.")
    elif v["kit"] == "yes":
        say("You did not take precautions, but you bought a repair kit, so you can continue your mission.")
    else:
        fail("You took no precautions and you have no repair kit. The damage is too severe to continue.")
    say("Oh no! A meteoroid has hit! It is officially a series of unexpected events. It caused no harm because it landed in a remote area, but the mission is getting harder for your mind.")
    say("You have been through so much, haven’t you? You need rest too. For astronauts living on Mars it is very important to feel happy, calm, and well, because being far from Earth and family for so long is hard. That is why both physical and mental health are taken seriously.")
    v["care"] = ask("Do you want psychological support and a health check?",
                    [("Yes, please 💙", "yes", ""), ("No, thanks", "no", "")])

    # ---- FINAL WEEKS
    banner("FINAL WEEKS")
    say("We are now in the final weeks of the mission. Your samples have been sent to Houston. The results should arrive in a few days.", "h")
    if v["garden"] == "small" and v["stock"] == "2.5":
        fail("Your supplies are low and your garden is small, so a food shortage is looming. We cannot continue the mission, but you came very close.")
    say("You did not face any food shortage, because your food stock and garden size were well balanced.")
    if v["care"] == "yes":
        say("Your test results came in today. Your blood pressure is high, but the necessary medication is included in the capsule.", "h")
    say("The systems have detected a possible dust storm. Hopefully some of your earlier decisions will help.")
    say("Dust storms happen when wind lifts fine sand and dust into the air. Sometimes they grow so large that they cover the entire planet and reduce sunlight.")
    if v["panel"] == "no":
        fail("We have to abandon the mission because you did not take the spare solar panels.")
    say("The panels now produce less energy, so we must activate the backup panels and adjust the life support system.")
    if v["power"] == "A":
        fail("With equal power levels you have plenty of oxygen, but your temperature and water supplies are at risk. We cannot continue under these conditions. Unfortunately, we are turning back.")
    say("Since we chose manual entry, we are switching the life support unit manually."
        if v["power"] == "C" else
        "The 45/30/25 split holds steady, so we are not intervening with the life support unit.")
    say("We are deploying our backup solar panels. The storm will pass!")

    # ---- SAMPLE DAY
    banner("SAMPLE DAY")
    say("Since you have the cold biological set, we will collect leaf samples from the garden."
        if v["tubes"] == "bio" else
        "Since you have the standard microtube set, we will collect a sample of Martian soil.")
    fill_tubes("leaf" if v["tubes"] == "bio" else "soil")
    say("Congratulations, you have filled the sample tubes! Your samples are on their way to Houston.", "h")
    say(f"You protected yourself from UV radiation, collected your test tubes, and helped humanity. You completed your mission, good job, {name}! But every good thing has an end. Now it is time to return, equipped with knowledge and having contributed to humanity...")
    scene("Mission complete")
    print("\n🎉 MISSION SUCCESSFULLY COMPLETED! 🎉")


def main():
    while True:
        try:
            play()
        except MissionFailed as e:
            print(f"\n[{WHO['h']}]")
            typewriter("MISSION FAILED. " + str(e))
        except (KeyboardInterrupt, EOFError):
            print("\nMission aborted.")
            return
        if input("\nPlay again? (y/n): ").strip().lower() != "y":
            print("Goodbye, astronaut! 🚀")
            return


if __name__ == "__main__":
    main()
