# Solo-Leveling-Text-Based-RPG-Game-in-C++

#include <iostream>
#include <random>
#include <ctime>
#include <algorithm>
#include <vector>
using namespace std;

class Player{
  // public:
  private:
  string name;
  float health, Defence, shieldDefence, armourDefence;
  int Melee_damage, Ranged_damage, MaxHealth, swordDamage, bowDamage;
  bool hasSword, hasShield, hasArmour, hasBow, hasCriticalHit, hasBlocker, hasLifeSteal, hasRangedAttack = false;
  int playerLevel = 1;
  bool usedRangedAttack = false;

  public:
  Player(float health, float Defence, int Melee_damage, int Ranged_damage): health(health),  Defence(Defence), Melee_damage(Melee_damage), Ranged_damage(Ranged_damage){};

  private:
  int takeDamagefromEmemy(int damage){
    if (damage == 0){//Blocker Activated
      return health;
    }
    else if (hasArmour){
      health = std::max(0.0f,health - (1-Defence-shieldDefence-armourDefence)*damage);
      return health;
    }
    else if(hasShield){
      health = std::max(0.0f,health - (1-Defence-shieldDefence)*damage);
      return health;
    }
    else{
      health = std::max(0.0f,health - (1-Defence)*damage);
      return health;
    }
  }

  bool isAlive(){
    return health > 0;
  }

  void levelUp() {
    playerLevel += 1;
    health = MaxHealth = max(MaxHealth * 1.1f, 100.0f);
    Defence += 0.05f;
    Melee_damage += 10;
    Ranged_damage += 5;
    cout << name << " leveled up to level " << playerLevel << "!" << endl;
  }

  public:
  friend class Game;
  ~Player() {}
};

class Enemy{
  // public:
  private:
  float health, Defence;
  int Attack;
  int GroundSlashDamage = 30;
  int SpeedDashDamage = 20;
  float LifeRegeneration = 0.2f;
  int maxBossHealth = 200;

  public:
  Enemy(float health, float Defence, int Attack): health(health), Defence(Defence), Attack(Attack){};

  private:
  int takeDamageFromPlayer(int damage){
    health = std::max(0.0f,health - (1-Defence)*damage);
    return health;
  }

  int giveDamage(){
    return Attack;
  }

  bool const isAlive() const{
    return health > 0;
  }
  public:
  friend class Game;
  Enemy() = default;
  ~Enemy() {};
};

class Game{
  public:
  // private:

  Player *player1 = nullptr;

  static int generateRandomInRange(int min, int max)
  {
    int randNum = rand() % (max - min + 1) + min;
    return randNum;
  }

  // public:
  // Game(Player *player1): player1(player1){};
  void start(){
    cout << "Welcome to Solo Leveling!" << endl;
    cout << "The Dragon Monarch awaits!\nDragon Monarch: Welcome to the showdown, Shadow Monarch!" << endl;
    cout << "What do you call yourself Monarch, in this life?: ";
  }

  void loop(){
    srand(time(0));
    Player player1(100, 0.1, 10, 5);
    player1.MaxHealth = player1.health;
    for ( player1.playerLevel = 1; player1.playerLevel <= 6;) {
      if(player1.isAlive()){
        switch (player1.playerLevel) {
          case 1: HandleLevels(player1); break;
          case 2: HandleLevels(player1); break;
          case 3: HandleLevels(player1); break;
          case 4: HandleLevels(player1); break;
          case 5: HandleLevels(player1); break;
          case 6: {
            Enemy boss(200, 0.3, 20);
            HandleLevels(player1);
            break;
          }
        }
      }
      // player1.levelUp();

      if (!player1.isAlive()) {
        break;
      } 
    }
  }

  private:

  std::vector<Enemy> create_enemies(int player_level, int num_enemies) {
    std::vector<Enemy> enemies;

    for (int i = 0; i < num_enemies; i++) {
      float health = 50 + (player_level * 5);
      float defence = 0.05 + (player_level * 0.01);
      int attack = 5 + (player_level * 2);

      if (player_level == 6 && i == 0) {
        health = 200;
        defence = 0.5;
        attack = 30;
      }

      enemies.push_back(Enemy(health, defence, attack)); 
    }
    return enemies;
  }

  void dealMeleeDamage(Player& player, std::vector<Enemy>& enemies) {
    int enemyChoice;
    std::cout << "Which dragon do you want to deal Melee damage to?" << std::endl;
    for (int i = 0; i < enemies.size(); ++i) {
      std::cout << i + 1 << ". " << "Dragon's Health: " << enemies[i].health << std::endl;
    }
    std::cin >> enemyChoice;

    Enemy& chosenEnemy = enemies[enemyChoice - 1];
    int damageDealt = player.Melee_damage;

    if (player.hasSword) {
        damageDealt += player.swordDamage;
    }

    if (player.hasCriticalHit){
      if (rand() % 10 == 0) {
        damageDealt *= 2;
        std::cout << "Critical hit!" << std::endl;
      }
    }

    chosenEnemy.takeDamageFromPlayer(damageDealt);

    if (player.hasLifeSteal){
      if (rand() % 10 == 0) {
        player.health += (0.1)*damageDealt;
        if (player.health > player.MaxHealth){
          player.health = player.MaxHealth;
        }
        std::cout << "Life steal activated! Your health has been restored by "<< std::min<float>((0.1f) * damageDealt, player.MaxHealth) << "."<< std::endl;
      }
    }

    if (!chosenEnemy.isAlive()) {
        enemies.erase(enemies.begin() + (enemyChoice - 1));
    }
  }

  void dealRangedDamage(Player& player, std::vector<Enemy>& enemies) {
    int damageDealt = player.Ranged_damage;
    if (player.hasBow) {
        damageDealt += player.bowDamage;
    }
    if (player.hasCriticalHit){
      if (rand() % 10 == 0) {
        damageDealt *= 2;
        std::cout << "Critical hit!" << std::endl;
      }
    }
    if (player.hasLifeSteal){
      if (rand() % 10 == 0) {
        player.health += (0.1)*damageDealt;
        if(player.health > player.MaxHealth){
          player.health = player.MaxHealth;
        }
        std::cout << "Life steal activated! Your health has been restored by "<< std::min<float>((0.1f) * damageDealt, player.MaxHealth) << "." << std::endl;
      }
    }
    if (player.hasRangedAttack){
      if (rand() % 10 == 0) {
        std::cout << "Special Ability Ruler's Authority Activated! You are invincible to the next attack of the Dragon Monarch!!" << std::endl;
        player.usedRangedAttack = true;
      }
    }

    for (Enemy& enemy : enemies) {
        enemy.takeDamageFromPlayer(damageDealt);
    }
  }

  void handleEnemyAttackCombined(Player& player, std::vector<Enemy>& enemies) {

    std::cout << "Dragon/s Attacked as well!!" << std::endl;
    int DamageDealt = 0;
    for (Enemy& enemy : enemies) {
      if(enemy.isAlive() && (player.playerLevel == 6)){
        int attackChoice = rand() % 3;
        switch (attackChoice) {
          case 0:
            DamageDealt += enemy.giveDamage();
            DamageDealt += enemy.GroundSlashDamage;
            std::cout << "Boss used Ground Slash!" << std::endl;
            if (player.usedRangedAttack){
              DamageDealt = 0;
              std::cout << "You won't get affected by this attack as Ruler's Authority is activated!" << std::endl;
              player.usedRangedAttack = false;
            }
            break;
          case 1:
            DamageDealt += enemy.giveDamage();
            DamageDealt += enemy.SpeedDashDamage;
            std::cout << "Boss used Speed Dash!" << std::endl;
            if (player.usedRangedAttack){
              DamageDealt = 0;
              std::cout << "You won't get affected by this attack as Ruler's Authority is activated!" << std::endl;
              player.usedRangedAttack = false;
            }
            break;
          case 2:
            enemy.health += (enemy.health) * (enemy.LifeRegeneration);  // Regenerate health
            if (enemy.health > enemy.maxBossHealth){
              enemy.health = enemy.maxBossHealth;
            }
            std::cout << "The Dragon regenerated 20% of its health!" << std::endl;
            std::cout << "After regenration, Dragon's current health is: " <<enemy.health << std::endl;
            break;
        }
      }
      else if (enemy.isAlive()) {
        DamageDealt += enemy.giveDamage();
      }
    }
    if (player.hasBlocker){
      if (rand() % 10 == 0) {
        DamageDealt = 0;
        std::cout << "Blocker Activated! You are invincible to the attack of the enemies." << std::endl;
      }
    }

    player.takeDamagefromEmemy(DamageDealt);

    if (player.health == 0) {
        PlayerHealth(player);
        std::cout << player.name << " is dead! Game Over!" << std::endl;
    }
  }

  void removeDeadEnemies(std::vector<Enemy>& enemies) {
      int aliveCount = 0;
      for (int i = 0; i < enemies.size(); ++i) {
          if (enemies[i].isAlive()) {
              if (i != aliveCount) {
                  std::swap(enemies[i], enemies[aliveCount]);
              }
              aliveCount++;
          }
      }
      enemies.resize(aliveCount);
  }


  void printPlayerStat(Player& player1){
    std::cout << player1.name<<"'s Health: " << player1.health <<std::endl;
    std::cout << player1.name<<"'s Melee Damage: " << player1.Melee_damage <<std::endl;
    std::cout << player1.name<<"'s Ranged Damage: " << player1.Ranged_damage <<std::endl;
    std::cout << player1.name<<"'s Defence: " << player1.Defence <<std::endl;
  }
  void PlayerHealth(Player& player1){
    std::cout << player1.name<<"'s Current Health: " << player1.health <<std::endl;
  }
  void printEnemyStat(const Enemy& enemy) {
    std::cout << "Dragon's Health: " << enemy.health << std::endl;
    std::cout << "Dragon's Attack: " << enemy.Attack << std::endl;
    std::cout << "Dragon's Defence: " << enemy.Defence << std::endl;
  }
  void EnemyHealth(const Enemy& enemy) {
    std::cout << "Dragon's Health: " << enemy.health << std::endl;
  }

  void playLevel(Player& player1)
  {
    std::vector<Enemy> enemies;
    if(player1.playerLevel < 6){
      std::cout << player1.name <<", Welcome to Level "<< player1.playerLevel <<"!" << std::endl;
      int numEnemies = player1.playerLevel;
      printPlayerStat(player1);

      enemies = create_enemies(player1.playerLevel, numEnemies);
      std::cout << "You have encountered " << enemies.size() << " Level" << player1.playerLevel <<" Dragon/s!" << std::endl;
      for (int i = 0; i < enemies.size(); ++i) {
        std::cout << "Dragon"<< i+1 <<":" << std::endl;
        printEnemyStat(enemies[i]);
      }
    }
    else if(player1.playerLevel == 6){
      printPlayerStat(player1);
      enemies = create_enemies(6, 1); // Create a single boss for level 6
      for (int i = 0; i < enemies.size(); ++i) {
        std::cout << "The Dragon Monarch:" << std::endl;
        printEnemyStat(enemies[i]);
      }
      std::cout << "But that's not it! \nFYI: The Dragon Monarch has hidden attacks like Ground Slash(dc = 30), Speed Dash(dc = 20) and Health Regenration(20%)!" << std::endl;
    }

    while(player1.isAlive() && !enemies.empty())
    { //player attack to enemies
      std::cout << "Choose the attack to the Dragon: 1.Melee 2.Ranged" << std::endl;
      int choice, DamageDealt;
      std::cin >> choice;
      switch(choice)
      {
        case 1:
          dealMeleeDamage(player1, enemies);
          for (int i = 0; i < enemies.size(); ++i) {
            EnemyHealth(enemies[i]);
          }
          break;
        default:
          dealRangedDamage(player1, enemies);
          for (int i = 0; i < enemies.size(); ++i) {
            EnemyHealth(enemies[i]);
          }
          break;
      }

      //Delete enemies if defeated
      removeDeadEnemies(enemies);

     //Enemy attack to player
      DamageDealt = 0;
      if (!enemies.empty()) {
        handleEnemyAttackCombined(player1, enemies);
        PlayerHealth(player1);
      }

      if (enemies.empty() && player1.playerLevel == 6) {
        std::cout << "All enemies defeated! You win Final level "<< player1.playerLevel <<"! \n Game Finished!!" << std::endl;
        std::cout << player1.name <<" is now Free!!!" << std::endl;
        std::cout << "---------------------------------------------------------------------" << std::endl;
        player1.playerLevel += 1;
      }
      else if (enemies.empty()) {
        std::cout << "All dragons defeated! You win level "<< player1.playerLevel <<"!" << std::endl;
        std::cout << player1.name <<" can now go to level "<< player1.playerLevel + 1 <<"!" << std::endl;
        player1.levelUp();
      }
    }
  }

  void HandleLevels(Player& player1) {
    for (int level = player1.playerLevel;level <= 6;) {
      // Level-specific actions:
      switch (player1.playerLevel) {
        case 1:
          std::cin >> player1.name;
          playLevel(player1);
          break;
        case 2:
          playLevel(player1);
          if (player1.isAlive()) {
            player1.hasSword = true;
            player1.hasCriticalHit = true;
            player1.swordDamage = 20;
            std::cout << "You have been awarded a special ability of Critical Hit(damage boost of 100%!!!), which has probability of activation = 10%. \nYou found a Dagger with Melee Attack damage of 20!!!" << std::endl;
          }
          break;
        case 3:
          playLevel(player1);
          if (player1.isAlive()) {
            player1.hasShield = true;
            player1.shieldDefence = 0.2;
            player1.hasBlocker = true;
            std::cout << "You have been awarded a special ability of Blocker(will get damage nulliefied on enemy hit!!!), which has probability of activation = 10%. \nYou found a Shield with additional defence boost of 10%!!!" << std::endl;
          }
          break;
        case 4:
          playLevel(player1);
          if (player1.isAlive()) {
            player1.hasArmour = true;
            player1.armourDefence = 0.1;
            player1.hasLifeSteal = true;
            std::cout << "You have been awarded a special ability of Life Steal(will get HP recovery on enemy hit!!!), which has probability of activation = 10%. \nYou found an Armer with additional defence boost of 10%!!!" << std::endl;
          }
          break;
        case 5:
          playLevel(player1);
          if (player1.isAlive()) {
            player1.hasBow = true;
            player1.hasRangedAttack = true;
            player1.bowDamage = 10;
            std::cout << "You have been awarded a special ability of Ruler's Authority(will get immune to the next attack of the Boss!), which has probability of activation = 10%. \nYou found an Bow with Ranged Attack damage of 10!!!" << std::endl;
          }
          break;
        case 6:
          Enemy boss(200, 0.3, 20);
          std::cout << "\n\n\n\n\n\n----------------------------------------------" << std::endl;
          std::cout << player1.name <<", Welcome to Final Dragon Monarch's lier!" << std::endl;
          std::cout << "The Dragon Monarch is waiting for you at the end of the lier! \n Defeat him if you can to get yourself free or DIE!!!" << std::endl;
          std::cout << "You have encountered The Dragon Monarch!" << std::endl;
          std::cout << "----------------------------------------------" << std::endl;
          playLevel(player1);
          break;
      }

      if (!player1.isAlive()) {
        std::cout << "Game Over!" << std::endl;
      }

    }
  }

  public:
  ~Game(){}
};

int main() {
  Game game;
  game.start();
  game.loop();
  return 0;
}
