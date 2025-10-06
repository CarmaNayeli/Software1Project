**Creature Sanctuary**
A creature collection and habitat restoration game with strategic type compatibility mechanics.

**Story:**
You are a newly hired Habitat Specialist at a once-renowned sanctuary. Years of neglect have left the sanctuary in ruins - the habitats are damaged, and all the creatures have fled into the surrounding wilderness.
Armed with your field journal and the sanctuary's old compatibility charts, you set out with a mission: find every creature species that once called the sanctuary home and convince them to return.

You begin your journey in the overgrown Meadow with a single lonely creature as your companion. But the creatures are wary of humans! They will only trust you if you bring along one of their compatible species as proof of your good intentions. Each exploration trip, you must choose carefully - you can only bring one creature companion with you. Your choice determines which of the wild creatures you encounter might be willing to come home with you.

Will you succeed in bringing every lost species home?

Outline 
1. Get your first creature and intro to the sanctuary 
          - one creature
          - four habitats that fit four creatures each

2. Gameplay Loop:
         - Choose a creature from the sanctuary to take with you,
         - be presented with three creatures, choose one to attempt to bring back, compatibility check between your creature and the wild creature, if success-> take back and select habitat for it and if not -> nothing happens,
         - if all creatures are happy, repeat. If not, you must rearrange or release a creature based on compatibility scores before you can explore again

3. Field Journal updates with new creature species, shows what you need to have in the sanctuary to progress to next area

Win condition: Field journal complete

This game uses a relational database to manage creatures, territories, and compatibility relationships.


**Database:**
This game uses its own database as made by the creature_schema.sql doc. It should be initialized by running setup-database.py and should not require any additional manual editing. 

The schema is, however, as follows:

-- MariaDB Schema for Simplified Creature Catcher Game
-- Database: creature_catcher

CREATE DATABASE IF NOT EXISTS creature_catcher;
USE creature_catcher;

-- Types table (16 unique types)
CREATE TABLE types (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT
);

-- Type relationships (effectiveness multipliers)
CREATE TABLE type_relationships (
    id INT PRIMARY KEY AUTO_INCREMENT,
    attacker_type_id INT NOT NULL,
    defender_type_id INT NOT NULL,
    multiplier DECIMAL(3,1) NOT NULL,
    FOREIGN KEY (attacker_type_id) REFERENCES types(id),
    FOREIGN KEY (defender_type_id) REFERENCES types(id),
    UNIQUE KEY unique_relationship (attacker_type_id, defender_type_id)
);

-- Creatures table (16 unique creatures)
CREATE TABLE creatures (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE,
    type_id INT NOT NULL,
    rarity ENUM('common', 'uncommon', 'rare', 'legendary') DEFAULT 'common',
    description TEXT,
    image_path VARCHAR(255) DEFAULT NULL,
    FOREIGN KEY (type_id) REFERENCES types(id)
);

-- Players table
CREATE TABLE players (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Habitats (each player can have multiple habitats)
CREATE TABLE habitats (
    id INT PRIMARY KEY AUTO_INCREMENT,
    player_id INT NOT NULL,
    habitat_number INT NOT NULL,
    FOREIGN KEY (player_id) REFERENCES players(id) ON DELETE CASCADE,
    UNIQUE KEY unique_player_habitat (player_id, habitat_number)
);

-- Player's captured creatures
CREATE TABLE player_creatures (
    id INT PRIMARY KEY AUTO_INCREMENT,
    player_id INT NOT NULL,
    creature_id INT NOT NULL,
    nickname VARCHAR(100),
    habitat_id INT DEFAULT NULL,
    habitat_slot INT DEFAULT NULL,
    happiness INT DEFAULT 70,
    caught_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (player_id) REFERENCES players(id) ON DELETE CASCADE,
    FOREIGN KEY (creature_id) REFERENCES creatures(id),
    FOREIGN KEY (habitat_id) REFERENCES habitats(id) ON DELETE SET NULL
);

-- Insert the 16 types
INSERT INTO types (name, description) VALUES
('Bloom', 'Natural and growing creatures'),
('Flame', 'Aggressive and burning creatures'),
('Metal', 'Metallic and sturdy creatures'),
('Shadow', 'Shadowy and mysterious creatures'),
('Mind', 'Mental and mystical creatures'),
('Earth', 'Earthy and grounded creatures'),
('Frost', 'Cold and freezing creatures'),
('Spirit', 'Ethereal and spectral creatures'),
('Tide', 'Fluid and aquatic creatures'),
('Thunder', 'Energetic and shocking creatures'),
('Wind', 'Aerial and free creatures'),
('Stone', 'Solid and defensive creatures'),
('Toxic', 'Poisonous and corrupting creatures'),
('Light', 'Radiant and pure creatures'),
('Fae', 'Mystical and enchanting creatures'),
('Myth', 'Powerful and legendary creatures');

-- Insert type relationships (effectiveness)
-- Format: attacker_type, defender_type, multiplier
-- 2.0 = super effective, 1.0 = neutral (not stored), 0.5 = not very effective

INSERT INTO type_relationships (attacker_type_id, defender_type_id, multiplier) VALUES
-- Bloom (id=1)
(1, 6, 2.0), (1, 9, 2.0), (1, 12, 2.0),  -- Strong against Earth, Tide, Stone
(1, 2, 0.5), (1, 7, 0.5), (1, 11, 0.5), (1, 13, 0.5),  -- Weak against Flame, Frost, Wind, Toxic

-- Flame (id=2)
(2, 1, 2.0), (2, 3, 2.0), (2, 7, 2.0), (2, 13, 2.0),  -- Strong against Bloom, Metal, Frost, Toxic
(2, 6, 0.5), (2, 9, 0.5), (2, 12, 0.5), (2, 15, 0.5),  -- Weak against Earth, Tide, Stone, Fae

-- Metal (id=3)
(3, 7, 2.0), (3, 12, 2.0), (3, 14, 2.0), (3, 15, 2.0),  -- Strong against Frost, Stone, Light, Fae
(3, 2, 0.5), (3, 5, 0.5), (3, 6, 0.5), (3, 10, 0.5), (3, 16, 0.5),  -- Weak against Flame, Mind, Earth, Thunder, Myth

-- Shadow (id=4)
(4, 5, 2.0), (4, 8, 2.0),  -- Strong against Mind, Spirit
(4, 14, 0.5), (4, 15, 0.5),  -- Weak against Light, Fae

-- Mind (id=5)
(5, 4, 2.0), (5, 13, 2.0),  -- Strong against Shadow, Toxic
(5, 3, 0.5), (5, 8, 0.5), (5, 16, 0.5),  -- Weak against Metal, Spirit, Myth

-- Earth (id=6)
(6, 2, 2.0), (6, 3, 2.0), (6, 10, 2.0), (6, 12, 2.0), (6, 13, 2.0),  -- Strong against Flame, Metal, Thunder, Stone, Toxic
(6, 1, 0.5), (6, 7, 0.5), (6, 9, 0.5),  -- Weak against Bloom, Frost, Tide

-- Frost (id=7)
(7, 1, 2.0), (7, 6, 2.0), (7, 11, 2.0), (7, 16, 2.0),  -- Strong against Bloom, Earth, Wind, Myth
(7, 2, 0.5), (7, 3, 0.5), (7, 9, 0.5), (7, 12, 0.5),  -- Weak against Flame, Metal, Tide, Stone

-- Spirit (id=8)
(8, 4, 2.0), (8, 5, 2.0),  -- Strong against Shadow, Mind
(8, 13, 0.5), (8, 14, 0.5), (8, 15, 0.5),  -- Weak against Toxic, Light, Fae

-- Tide (id=9)
(9, 2, 2.0), (9, 6, 2.0), (9, 12, 2.0),  -- Strong against Flame, Earth, Stone
(9, 1, 0.5), (9, 10, 0.5), (9, 16, 0.5),  -- Weak against Bloom, Thunder, Myth

-- Thunder (id=10)
(10, 3, 2.0), (10, 9, 2.0), (10, 11, 2.0),  -- Strong against Metal, Tide, Wind
(10, 6, 0.5), (10, 12, 0.5),  -- Weak against Earth, Stone

-- Wind (id=11)
(11, 1, 2.0), (11, 5, 2.0), (11, 13, 2.0),  -- Strong against Bloom, Mind, Toxic
(11, 7, 0.5), (11, 10, 0.5), (11, 12, 0.5),  -- Weak against Frost, Thunder, Stone

-- Stone (id=12)
(12, 2, 2.0), (12, 7, 2.0), (12, 10, 2.0), (12, 11, 2.0), (12, 13, 2.0),  -- Strong against Flame, Frost, Thunder, Wind, Toxic
(12, 1, 0.5), (12, 3, 0.5), (12, 6, 0.5), (12, 9, 0.5),  -- Weak against Bloom, Metal, Earth, Tide

-- Toxic (id=13)
(13, 1, 2.0), (13, 5, 2.0), (13, 15, 2.0),  -- Strong against Bloom, Mind, Fae
(13, 2, 0.5), (13, 3, 0.5), (13, 6, 0.5), (13, 11, 0.5), (13, 12, 0.5),  -- Weak against Flame, Metal, Earth, Wind, Stone

-- Light (id=14)
(14, 4, 2.0), (14, 5, 2.0), (14, 8, 2.0),  -- Strong against Shadow, Mind, Spirit
(14, 3, 0.5), (14, 16, 0.5),  -- Weak against Metal, Myth

-- Fae (id=15)
(15, 2, 2.0), (15, 4, 2.0), (15, 8, 2.0), (15, 13, 2.0), (15, 16, 2.0),  -- Strong against Flame, Shadow, Spirit, Toxic, Myth
(15, 3, 0.5),  -- Weak against Metal

-- Myth (id=16)
(16, 3, 2.0), (16, 5, 2.0), (16, 9, 2.0), (16, 11, 2.0), (16, 14, 2.0), (16, 16, 2.0),  -- Strong against Metal, Mind, Tide, Wind, Light, Myth
(16, 7, 0.5), (16, 15, 0.5);  -- Weak against Frost, Fae

-- Insert the 16 creatures (one per type) with image paths in images/ folder
INSERT INTO creatures (name, type_id, rarity, description, image_path) VALUES
('Spriggle', 1, 'common', 'A sprightly plant creature with verdant leaves', 'images/spriggle.png'),
('Capychara', 2, 'uncommon', 'A capybara-like creature wreathed in flames', 'images/capychara.png'),
('Chromutt', 3, 'uncommon', 'A metallic dog with a chrome-plated body', 'images/chromutt.png'),
('Obscurine', 4, 'rare', 'A shadowy feline that blends into darkness', 'images/obscurine.png'),
('Pandascend', 5, 'rare', 'A mystical panda with psychic powers', 'images/pandascend.png'),
('Granidam', 6, 'common', 'A sturdy creature made of earth and stone', 'images/granidam.png'),
('Borealynx', 7, 'rare', 'An elegant lynx with icy fur and frost breath', 'images/borealynx.png'),
('Reverwing', 8, 'rare', 'A spectral bird with ethereal wings', 'images/reverwing.png'),
('Marinlet', 9, 'common', 'A small aquatic creature with flowing fins', 'images/marinlet.png'),
('Pangostrike', 10, 'uncommon', 'A pangolin that crackles with electricity', 'images/pangostrike.png'),
('Cirroptera', 11, 'uncommon', 'A bat-like creature that rides the wind', 'images/cirroptera.png'),
('Geckathyst', 12, 'common', 'A gecko with an amethyst-encrusted hide', 'images/geckathyst.png'),
('Shaduran', 13, 'common', 'A venomous lizard with toxic secretions', 'images/shaduran.png'),
('Cosmeow', 14, 'rare', 'A radiant cat that glows with inner light', 'images/cosmeow.png'),
('Gemwraith', 15, 'rare', 'A mystical fae creature adorned with gems', 'images/gemwraith.png'),
('Starsprit', 16, 'legendary', 'A cosmic sprite born from starlight and myth', 'images/starsprit.png');
