# 3D Game Programming & Design：Game Intelligence

# Game Intelligence Overview
Game intelligence is:
> Under the constraints of the rules of the game, the Non-Player Character (NPC) in the game is presented as a game opponent with certain human intelligent behavior through the appropriate algorithm, so that the game players face continuous challenges, and gain from the challenges, including knowledge and skills.

Game intelligence needs to have the following qualities:

 - Personified: Game intelligence focuses on game objects that behave like (simulated) people, both doing surprisingly effective things and making all sorts of silly mistakes to match different people in the real world. Artificial intelligence, on the other hand, is often superhuman in design, seeking the best results.
 - Playability: Game intelligence does not mean a grand algorithm, it focuses more on the design of NPCS with different abilities for different types of players, such as "small monsters", "big bosses" and so on. Usually, the parameter level is used to express the ability of game intelligence agent.
 - Interesting: Intelligent game design focuses on entertainment, not just algorithmic research. As a result, the algorithm will assemble some special effects that make people feel happy.

# Models, Methods and Common Algorithms
**The "Sense-Think-Act" Paradigm**

"Sense-Think-Act" paradigm is the basic concept for constructing agent, robot and NPC. Since it was proposed in the 1980s, we have used the "Sense-Think-Act" paradigm to think about how robots work and design them. 

**1.Sense**

Perception is the behavior of agent receiving world information, and the data it obtains will be the input of thinking. The main problem to be considered here is that how to limit information acquisition is the core problem of agent design at different levels.

In games, the ability to acquire information is often defined in terms of visual, auditory, and olfactory channels:

 - Vision
 	Identify "enemy" position and attributes Identify "obstacles" and their range.
     
 - Sound 
 	Identify the direction and distance of the event.
     
 - Smell
   Get traces of the player/event.

It's common to use distance, Angle, obstacle to restrict agent to find information such as player's position, or to use interference to affect the accuracy of information. If agents all have the player's location and navigation map, the result will be a swarm of agents, which is not fun. Appropriate restrictions will enable agents to present rich behaviors under the same decision algorithm.

A simple method is that agent combines different types of triggers and probes to construct different information acquisition scenes. Finally put perception results in a data structure (similar to UnityEngine.EventSystems.PointerEventData).

 **When Agent Thinking, data information other than perception cannot be obtained.**

**2.Think**

Think is an algorithm whose inputs are perceived data and outputs are behaviours. The algorithm of thinking is usually part of what we call the rules of the game, what the agent can and should do. The game agent's thinking is similar to the decision-making process of the human brain, and establishes behaviors that conform to the difficulty curve of the game player and can be controlled and conform to social norms. Another related issue is that the player difficulty curve is unknown at the programming stage and depends on the outcome of the actions and confrontations of many players and agents. There are huge uncertainties in playtesting and operation. Therefore, it's not feasible to write the agent-decision-making process using "If... Then..." sentence.

At present, the main methods are:

**Rule-based Inference Engine**

Rule-based Inference Engine is also called Inference Model，It's developed from rule-based expert systems. The basic concepts of rule-based expert system include:

 - Facts: Facts are used to represent known data or information.
 - Rules: Rules are production rules, used to express the knowledge of system reasoning.
   Rules consist of conditions and actions in the format of：IF CONDITION Then ACT，eg. Rule1: Human(x) => Mortal(x)

**State Machine Engine**

Finite State Machine / FSM）is a graphical automatic execution tool. In Unity, it is a standardized agent (NPC) action control tool. The main tasks of defining a state machine include:

- States: This component defines a set of states that a game entity or NPC can choose from (patrol, chase, and shoot)
- Transition: This component defines the relationship between different states
- Rules: This component is used to trigger state transitions (players are in line of sight, far enough away to attack, lose/kill players)
- Events: This component is used to trigger check rules (guard visible area, distance from player, etc.)

**Decision Trees**

Decision Trees are also known as Behaviour Trees.

**3. Act**

Act takes the result of Think as the input. The task of this part is to make the agent's behavior more consistent with the laws of the physical world, so that the ideal result of "getting what you want" becomes uncertain.

Common elements to be considered include:

- Preparation time. In the preparation time by means of light, sound, etc
- Action time. From the beginning to the end of the action is a sequence
- Interference factors. The use of wind, terrain, random number to make the cannon ball has a certain deviation

# Unity 3D Navigation and Pathfinding
Unity Navigation systems allow the creation of game worlds for navigating game characters. The game character can find the shortest path to any point on the blue unicom grid, and has the ability to climb slopes and jump gullies. 

- **NavMesh** (Navigation Mesh) is a data structure that describes the surface on which a game object can walk. Through the triangular mesh, calculate the shortest path between any two points, used for game object navigation. It is automatically created or baked based on scene geometry.
-
- **NavMesh Agent** component creates a role with pathfinding capability. Agents use NavMesh reasoning to avoid each other and move obstacles.
- **Off-Mesh** component allows "portals" to be established between disconnected blocks. For example, jumping over a ditch or fence, or opening a door before going through it, can be described as an off-mesh Link.
- **NavMesh Obstacle** components allow you to describe agents. Obstacles should be avoided when moving. Buckets or boxes controlled by physical systems are typical obstacles. While the obstacle is moving, the Agent tries to avoid it, but once the obstacle becomes stationary, it will cut a hole in the navigation grid so that the Agent can change their path to bypass it, or the stationary obstacle blocks the path, allowing the Agent to find another route.

# Programming Practice
## AI design of Tanks Battle Game

- Model the AI tank using the "sense-think-act" paradigm
- Put some obstacles in the scene to block your opponent's view
- The tank needs to place a matrix bounding box trigger so that the AI tank can use ray to detect the opponent's position
- AI tanks only use navigation when targets are visible and be able to avoid obstacles.
- Realize human-computer battle

## Programming Analysis
Add text to implement various tank movements, open "TankMovement" script and edit it. Initialization of HP and control of how the tank's HP changes after being hit by bullets and how it fires bullets.

```javascript
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Tank : MonoBehaviour {
    private float hp =1000.0f;
    // Initialization
    public Tank()
    {
        hp = 1000.0f;
    }

    public float getHP()
    {
        return hp;
    }

    public void setHP(float hp)
    {
        this.hp = hp;
    }

    public void beShooted()
    {
        hp -= 100;
    }
    
    public void shoot(TankType type)
    {
        GameObject bullet = Singleton<MyFactory>.Instance.getBullets(type);
        bullet.transform.position = new Vector3(transform.position.x, 2.0f, transform.position.z) + transform.forward * 2.0f;
        bullet.transform.forward = transform.forward; // Direction
        bullet.GetComponent<Rigidbody>().AddForce(bullet.transform.forward * 20, ForceMode.Impulse);
    }
}

```
AI tank, also known as the enemy tank, is one of the key points of this program. Here I use the principles and techniques described in the previous sections to realize the automatic cruise of the NPC to locate the player's position and automatically approach the player's position.

Navigation utilizes the NavMeshAgent Agent component to create AI tanks with pathfinding capabilities. Agents use NavMesh reasoning to avoid collisions and move obstacles. In addition, we use coroutines for auto-shooting, firing bullets when within 10 of the player.

```javascript
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class Enemy : Tank {
    public delegate void RecycleEnemy(GameObject enemy);
    // When enemy is destroyed, note the factory to recycle it;
    public static event RecycleEnemy recycleEnemy;
    // position of player
    private Vector3 playerLocation;
    private bool gameover;
    private void Start()
    {
        playerLocation = GameDirector.getInstance().currentSceneController.getPlayer().transform.position;
        StartCoroutine(shoot());
    }

    void Update() {
        playerLocation = GameDirector.getInstance().currentSceneController.getPlayer().transform.position;
        gameover = GameDirector.getInstance().currentSceneController.getGameOver();
        if (!gameover)
        {
            if (getHP() <= 0 && recycleEnemy != null)
            {
                recycleEnemy(this.gameObject);
            }
            else
            {
                // Automatically move to player
                NavMeshAgent agent = gameObject.GetComponent<NavMeshAgent>();
                agent.SetDestination(playerLocation);
            }
        }
        else
        {
            // The game is over. Stop pathfinding
            NavMeshAgent agent = gameObject.GetComponent<NavMeshAgent>();
            agent.velocity = Vector3.zero;
            agent.ResetPath();
        }
    }
    // Coroutines implement shooting every 1s. Begin to love coroutines!
    IEnumerator shoot()
    {
        while (!gameover)
        {
            for(float i =1;i> 0; i -= Time.deltaTime)
            {
                yield return 0;
            }
            if(Vector3.Distance(playerLocation,gameObject.transform.position) < 10)
            {
                shoot(TankType.ENEMY);
            }
        }
    }
}

```

The player tank is the inheritance of the tank class, because the player needs us to control its shooting, direction, and movement, so here's all about that.

W/S control forward and backward, and A/D control rotation of left and right.

```javascript
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Player : Tank{
    public delegate void DestroyPlayer(); // game over!
    public static event DestroyPlayer destroyEvent;
    void Start () {
        setHP(1000);
	}
	
	// Update is called once per frame
	void Update () {
		if(getHP() <= 0)    // game over!
        {
            this.gameObject.SetActive(false);
            destroyEvent();
        }
	}

    // W/S control forward and backward
    public void moveForward()
    {
        gameObject.GetComponent<Rigidbody>().velocity = gameObject.transform.forward * 50;
    }
    public void moveBackWard()
    {
        gameObject.GetComponent<Rigidbody>().velocity = gameObject.transform.forward * -50;
    }

    // A/D control the rotation of left and right on the spot. By changing the euler Angle of the player's tank with increments on the horizontal axis, the tank turns
    public void turn(float offsetX)
    {
        float x = gameObject.transform.localEulerAngles.x;
        float y = gameObject.transform.localEulerAngles.y + offsetX*2;
        gameObject.transform.localEulerAngles = new Vector3(x, y, 0);
    }
}

```
First of all, the bullet class is to distinguish the attack is the enemy or their own side, so I first make a judgment on the tank type, if the enemy will attack, otherwise they will not shoot.

At the same time, in order to make the game more playable, I set the player's attack power to be stronger than the AI tank, reducing the difficulty of the game.

```javascript
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Bullet : MonoBehaviour {
    public float explosionRadius = 3.0f;
    private TankType tankType;

    // Set the type of tank that fires bullets
    public void setTankType(TankType type)
    {
        tankType = type;
    }
    private void OnCollisionEnter(Collision collision)
    {
        if(collision.transform.gameObject.tag == "tankEnemy" && this.tankType == TankType.ENEMY ||
            collision.transform.gameObject.tag == "tankPlayer" && this.tankType == TankType.PLAYER)
        {
            return;
        }
        MyFactory factory = Singleton<MyFactory>.Instance;
        ParticleSystem explosion = factory.getParticleSystem();
        explosion.transform.position = gameObject.transform.position;
        // Get all colliders within the range of explosion
        Collider[] colliders = Physics.OverlapSphere(gameObject.transform.position, explosionRadius);
        
        foreach(var collider in colliders)
        {
            // The distance between the tank being hit and the center of explosion
            float distance = Vector3.Distance(collider.transform.position, gameObject.transform.position);
            float hurt;
            // The damage is a littel higher if the bullet is fired by the player
            if (collider.tag == "tankEnemy" && this.tankType == TankType.PLAYER)
            {
                hurt = 300.0f / distance;
                collider.GetComponent<Tank>().setHP(collider.GetComponent<Tank>().getHP() - hurt);
            }
            else if(collider.tag == "tankPlayer" && this.tankType == TankType.ENEMY)
            {
                hurt = 100.0f / distance;
                collider.GetComponent<Tank>().setHP(collider.GetComponent<Tank>().getHP() - hurt);
            }
            explosion.Play();
        }

        if (gameObject.activeSelf)
        {
            factory.recycleBullet(gameObject);
        }
    }

}

```
Then the factory class is to produce and recycle the roles above, which I used a lot earlier. The scene class is an instantiation process that initializes and implements the actions of the UI class. The UI class is about interaction with the user, monitoring the player's input and output of the game and so on.

## Effect of Game Interface
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191205185955987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDM3NzY5MQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191205190026117.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDM3NzY5MQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120519004624.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDM3NzY5MQ==,size_16,color_FFFFFF,t_70)

- At last:

[Demonstration Video](https://cscs1999.github.io/Tanks-Battle-Game/)

