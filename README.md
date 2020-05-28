# oop-repository-hashir
object oriented programming
Tappy bird:
CODE
#BACKGROUND COLLIDER SCRIPT:
using UnityEngine;

public class BackgroundCollideController : MonoBehaviour
{
    private int numberOfBackgrounds;
    private float distanceBetweenBackgrounds;

    private int numberOfGrounds;
    private float distanceBetweenGrounds;

    private int numberOfPipes;
    private float distanceBetweenPipes;

    private bool upperPipe;

    public void Start()
    {
        var backgrounds = GameObject.FindGameObjectsWithTag("Background");
        var grounds = GameObject.FindGameObjectsWithTag("Ground");
        var pipes = GameObject.FindGameObjectsWithTag("Pipe");

        RandomizePipes(pipes);

        this.numberOfBackgrounds = backgrounds.Length;
        this.numberOfGrounds = grounds.Length;
        this.numberOfPipes = pipes.Length;

        if (this.numberOfBackgrounds < 2 
            || this.numberOfGrounds < 2
            || this.numberOfPipes < 2)
        {
            throw new System.InvalidOperationException("You must have at least two backgrounds or grounds or pipes in your scene!");
        }

        this.distanceBetweenBackgrounds = this.DistanceBetweenObjects(backgrounds);
        this.distanceBetweenGrounds = this.DistanceBetweenObjects(grounds);
        this.distanceBetweenPipes = this.DistanceBetweenObjects(pipes);
    }

    public void OnTriggerEnter2D(Collider2D collider)
    {
        if (collider.CompareTag("Background") 
            || collider.CompareTag("Ground") 
            || collider.CompareTag("Pipe"))
        {
            var go = collider.gameObject;
            var originalPosition = go.transform.position;

            if (collider.CompareTag("Background"))
            {
                originalPosition.x +=
                    this.numberOfBackgrounds
                    * this.distanceBetweenBackgrounds;
            }
            else if (collider.CompareTag("Ground"))
            {
                originalPosition.x +=
                    this.numberOfGrounds
                    * this.distanceBetweenGrounds;
            }
            else
            {
                originalPosition.x +=
                    this.numberOfPipes
                    * this.distanceBetweenPipes;

                float randomY;
                if (this.upperPipe)
                {
                    randomY = Random.Range(1.5f, 3);
                }
                else
                {
                    randomY = Random.Range(-1, 0.5f);
                }
                originalPosition.y = randomY;

                this.upperPipe = !this.upperPipe;
            }

            go.transform.position = originalPosition;
        }
    }

    private float DistanceBetweenObjects(GameObject[] gameObjects)
    {
        float minDistance = float.MaxValue;

        for (int i = 1; i < gameObjects.Length; i++)
        {
            var currentDistance = Mathf.Abs(
                gameObjects[i - 1].transform.position.x
                - gameObjects[i].transform.position.x);

            if (currentDistance < minDistance)
            {
                minDistance = currentDistance;
            }
        }

        return minDistance;
    }

    private void RandomizePipes(GameObject[] pipes)
    {
        int count = 0;

        for (int i = 1; i < pipes.Length; i++)
        {
            count++;
            var currentPipe = pipes[i];
            float randomY;
            if (count % 2 == 0) // upper pipe
            {
                randomY = Random.Range(1.5f, 3);
            }
            else // down pipe
            {
                randomY = Random.Range(-1, 0.5f);
            }
            
            var pipePosition = currentPipe.transform.position;
            pipePosition.y = randomY;
            currentPipe.transform.position = pipePosition;
        }
    }
}
#BIRD CONTROLLER
using UnityEngine;

public class BirdController : MonoBehaviour
{
    public float flapSpeed = 1000f;
    public float maxFlapSpeed = 100f;
    public float forwardSpeed = 100f;

    public AudioClip flapSound;

    private Rigidbody2D rb;
    private Animator animator;
    private AudioSource audioSource;

    private bool didFlap;
    private bool isDead;

    private bool gameStarted;

    private Vector2 originalPosition;

    private GameObject startButton;

    private int score = 0;

    public void Start()
    {
        this.rb = this.GetComponent<Rigidbody2D>();
        this.animator = this.GetComponent<Animator>();
        this.audioSource = this.GetComponent<AudioSource>();

        this.startButton = GameObject.Find("StartButton");
        this.originalPosition = new Vector2(
            this.transform.position.x,
            this.transform.position.y);

        this.rb.gravityScale = 0;
        this.forwardSpeed = 0;
        this.animator.enabled = false;
    }

    // read input, change graphics
    public void Update()
    {
        if (Input.GetButtonDown("Fire1"))
        {
            if (!isDead)
            {
                if (!this.gameStarted)
                {
                    var renderer = startButton.GetComponent<SpriteRenderer>();
                    renderer.enabled = false;
                    this.forwardSpeed = 5;
                    this.rb.gravityScale = 1;
                    this.animator.enabled = true;
                }

                this.didFlap = true;
                this.audioSource.PlayOneShot(this.flapSound);
            }
            else
            {
                Application.LoadLevel("Play");
            }
        }
    }

    // apply physics
    public void FixedUpdate()
    {
        var velocity = this.rb.velocity;
        velocity.x = this.forwardSpeed;
        this.rb.velocity = velocity;

        if (this.rb.velocity.y > 0)
        {
            this.rb.MoveRotation(30);
        }
        else if (!isDead)
        {
            var angle = velocity.y * 8;
            if (angle < -90)
            {
                angle = -90;
            }
            this.rb.MoveRotation(angle);
        }

        if (didFlap)
        {
            didFlap = false;
            this.rb.AddForce(new Vector2(0, flapSpeed), ForceMode2D.Impulse);

            var updatedVelocity = this.rb.velocity;
            if (updatedVelocity.y > this.maxFlapSpeed)
            {
                updatedVelocity.y = this.maxFlapSpeed;
                this.rb.velocity = updatedVelocity;
            }
        }
    }

    public void OnCollisionEnter2D(Collision2D collider)
    {
        if (collider.gameObject.CompareTag("Floor") || collider.gameObject.CompareTag("PipeCollision"))
        {
            this.isDead = true;
            this.animator.SetBool("BirdDead", true);
            this.forwardSpeed = 0;

            var currentHighScore = PlayerPrefs.GetInt("HighScore", 0);

            if (this.score > currentHighScore)
            {
                PlayerPrefs.SetInt("HighScore", this.score);
            }

            var renderer = startButton.GetComponent<SpriteRenderer>();
            renderer.enabled = true;

            var startButtonX = Camera.main.transform.position.x;
            var startButtonY = Camera.main.transform.position.y;

            var startButtonPosition = this.startButton.transform.position;
            startButtonPosition.x = startButtonX;
            startButtonPosition.y = startButtonY;
            this.startButton.transform.position = startButtonPosition;
        }
    }

    public void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Pipe"))
        {
            this.score++;
            Debug.Log(score);
        }
    }
}



