# Live Coding: Football World

## Goal
Build a football system demonstrating **composition** (Stadium owns its parts) and **aggregation** (Teams have Players) side by side.

---

## Part A: Composition — Stadium

## Step 1: Pitch and Seats (parts of a stadium)

```python
class Pitch:  # Aikštė
    def __init__(self, surface):
        self.surface = surface

    def __str__(self):
        return f"Pitch({self.surface})"


class Seats:  # Sėdynės
    def __init__(self, capacity):
        self.capacity = capacity

    def __str__(self):
        return f"Seats({self.capacity})"
```

## Step 2: Stadium creates its own parts

```python
class Stadium:  # Stadionas
    def __init__(self, name, capacity, surface="grass"):
        self.name = name
        # Composition: created INSIDE — exclusive to this stadium
        self.pitch = Pitch(surface)
        self.seats = Seats(capacity)

    def info(self):
        print(f"Stadium: {self.name}")
        print(f"  {self.pitch}")
        print(f"  {self.seats}")
```

## Step 3: Try it

```python
camp_nou = Stadium("Camp Nou", 99354)
camp_nou.info()
```

### Output:
```
Stadium: Camp Nou
  Pitch(grass)
  Seats(99354)
```

> **Key point:** Pitch and Seats are created *inside* Stadium.
> Delete Camp Nou → pitch and seats are gone. That's composition!

---

## Part B: Aggregation — Teams and Players

## Step 4: Player

```python
class Player:  # Žaidėjas
    def __init__(self, name, position):
        self.name = name
        self.position = position

    def __str__(self):
        return f"{self.name}, Position: {self.position}"
```

## Step 5: Team (base class)

```python
class Team:  # Komanda
    def __init__(self, name):
        self.name = name
        self.players = []

    def add_player(self, player):
        self.players.append(player)

    def list_players(self):
        print(f"Team: {self.name}")
        for player in self.players:
            print(f"  {player}")

    def remove_player(self, player_to_remove):
        self.players = [player for player in self.players
                        if player.name != player_to_remove.name]
```

## Step 6: FootballTeam and NationalTeam

```python
class FootballTeam(Team):  # Futbolo komanda
    pass


class NationalTeam(Team):  # Nacionalinė komanda
    pass
```

> **Discussion:** Why inheritance here? FootballTeam *is-a* Team, NationalTeam *is-a* Team.

## Step 7: Create players (independent objects)

```python
player1 = Player("Leo Messi", "Forward")
player2 = Player("Cristiano Ronaldo", "Forward")
player3 = Player("Luis Figo", "Forward")
```

> **Key point:** Players are created *outside* any team — this is aggregation!

## Step 8: Create club teams and add players

```python
fc_barcelona = FootballTeam("FC Barcelona")
fc_barcelona.add_player(player1)
fc_barcelona.add_player(player3)

real_madrid = FootballTeam("Real Madrid")
real_madrid.add_player(player2)
```

## Step 9: Same players on national teams

```python
argentina = NationalTeam("Argentina")
argentina.add_player(player1)

portugal = NationalTeam("Portugal")
portugal.add_player(player2)
portugal.add_player(player3)
```

> **Key point:** Messi is on both FC Barcelona AND Argentina.
> The same object referenced by multiple teams — only possible with aggregation!

## Step 10: List players

```python
real_madrid.list_players()
fc_barcelona.list_players()
```

### Output:
```
Team: Real Madrid
  Cristiano Ronaldo, Position: Forward

Team: FC Barcelona
  Leo Messi, Position: Forward
  Luis Figo, Position: Forward
```

## Step 11: Transfer Luis Figo to Real Madrid

```python
fc_barcelona.remove_player(player3)
real_madrid.add_player(player3)

print("\n--- After transfer ---")
fc_barcelona.list_players()
real_madrid.list_players()
```

### Output:
```
--- After transfer ---
Team: FC Barcelona
  Leo Messi, Position: Forward

Team: Real Madrid
  Cristiano Ronaldo, Position: Forward
  Luis Figo, Position: Forward
```

## Step 12: Prove aggregation — delete a team

```python
del fc_barcelona
print(player1)  # Leo Messi, Position: Forward — still exists!
```

> The team is gone, but Messi lives on.

---

## Comparison

| | Stadium & Pitch | Team & Player |
|---|---|---|
| **Type** | Composition | Aggregation |
| **Created** | Inside `__init__` | Outside, passed in |
| **Shared?** | No — exclusive | Yes — multiple teams |
| **Delete owner** | Parts gone | Players survive |
| **Transfer?** | No — makes no sense | Yes — remove + add |

## Discussion Points
- Why is Pitch **composition** but Player **aggregation**? (A pitch belongs to one stadium; a player can play for multiple teams)
- Can you move a Pitch to another Stadium? (No — that's the point of composition)
- Where is **inheritance** in this example? (FootballTeam and NationalTeam inherit from Team)
- The transfer is just **remove + add** — the player object never changes, only the references do
