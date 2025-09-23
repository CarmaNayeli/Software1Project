Sanctuary
A creature collection and habitat restoration game with strategic type compatibility mechanics.
Story
You are a newly hired Habitat Specialist at a once-renowned sanctuary. Years of neglect have left the sanctuary in ruins - the habitats are damaged, and all the creatures have fled into the surrounding wilderness.
Armed with your field journal and the sanctuary's old compatibility charts, you set out with a mission: find every creature species that once called the sanctuary home and convince them to return.

You begin your journey in the overgrown Meadow with a single lonely creature as your companion. The old sanctuary map shows several locked territories beyond: mysterious forests, crystal caves, storm-swept peaks, and frozen waterfalls - each home to different creature types.
But the creatures are wary of humans! They will only trust you if you bring along one of their compatible species as proof of your good intentions. Each exploration trip, you must choose carefully - you can only bring one creature companion with you. Your choice determines which of the wild creatures you encounter might be willing to come home with you.

As you catalog more species in your field journal, new territories unlock, revealing rarer and more exotic creatures. Some creatures possess dual types, making them incredibly valuable for attracting multiple species - but also challenging to house harmoniously, as not all creatures get along! The compatibility charts reveal which types enhance each other's happiness and which combinations lead to stress and conflict. Building successful habitats requires both this knowledge and careful planning.

Will you succeed in bringing every lost species home?

Database
This game uses a relational database to manage creatures, territories, and compatibility relationships.

Create a new database 'game': CREATE DATABASE game;
Switch to that database: USE game;
Create the following tables:

sql   CREATE TABLE territories (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(50) NOT NULL,
       description TEXT,
       unlock_requirement INT DEFAULT 0
   );
sql   CREATE TABLE creatures (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(50) NOT NULL,
       type1 VARCHAR(20) NOT NULL,
       type2 VARCHAR(20) NULL,
       territory_id INT,
       rarity INT DEFAULT 1,
       description TEXT,
       FOREIGN KEY (territory_id) REFERENCES territories(id)
   );
sql   CREATE TABLE type_compatibility (
       id INT AUTO_INCREMENT PRIMARY KEY,
       type1 VARCHAR(20) NOT NULL,
       type2 VARCHAR(20) NOT NULL,
       multiplier DECIMAL(3,2) NOT NULL
   );
sql   CREATE TABLE game (
       id INT AUTO_INCREMENT PRIMARY KEY,
       player_name VARCHAR(40) NOT NULL,
       current_territory INT DEFAULT 1,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       FOREIGN KEY (current_territory) REFERENCES territories(id)
   );
sql   CREATE TABLE player_creatures (
       id INT AUTO_INCREMENT PRIMARY KEY,
       game_id INT NOT NULL,
       creature_id INT NOT NULL,
       nickname VARCHAR(50),
       caught_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       habitat_id INT DEFAULT 1,
       FOREIGN KEY (game_id) REFERENCES game(id),
       FOREIGN KEY (creature_id) REFERENCES creatures(id)
   );
sql   CREATE TABLE field_journal (
       id INT AUTO_INCREMENT PRIMARY KEY,
       game_id INT NOT NULL,
       creature_id INT NOT NULL,
       discovered_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       FOREIGN KEY (game_id) REFERENCES game(id),
       FOREIGN KEY (creature_id) REFERENCES creatures(id)
   );

Insert the type compatibility data:

sql   INSERT INTO type_compatibility (type1, type2, multiplier) VALUES
   ('Flame', 'Bloom', 2.0), ('Flame', 'Shadow', 2.0),
   ('Flame', 'Tide', 0.5), ('Flame', 'Frost', 0.5),
   ('Tide', 'Flame', 2.0), ('Tide', 'Stone', 2.0),
   ('Tide', 'Storm', 0.5), ('Tide', 'Shadow', 0.5),
   ('Stone', 'Storm', 2.0), ('Stone', 'Light', 2.0),
   ('Stone', 'Tide', 0.5), ('Stone', 'Shadow', 0.5),
   -- ... (continue for all type relationships)
   -- All unlisted combinations default to 1.0 (neutral)
   ;

Check that you have the necessary tables: SHOW TABLES;

The result should be like this:



text   +-------------------+
   | Tables_in_game    |
   +-------------------+
   | creatures         |
   | field_journal     |
   | game              |
   | player_creatures  |
   | territories       |
   | type_compatibility|
   +-------------------+
