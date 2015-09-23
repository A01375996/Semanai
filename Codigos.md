# Semanai

Codigo Scroll

public class Scroll : MonoBehaviour {

    public float speed = 0.5f;
    
    public Renderer rend;

    // Use this for initialization
    void Start () {
        rend = GetComponent < Renderer >();
	}
	
	// Update is called once per frame
	void Update () {

        Vector2 offset = new Vector2(Time.time * speed, 0);

        rend.material.mainTextureOffset = offset;


	}
}   

--------------------------------------------------------------------------------------------------------------------------------
Collectables

using UnityEngine;
using System.Collections;

public class Collectables : MonoBehaviour
{
    private AudioSource source;
    public AudioClip thud;

    void Start()
    {
        source = GetComponent<AudioSource>();
    }
    void OnTriggerEnter2D(Collider2D target)
    {
        if (target.gameObject.tag == "Player")
            Destroy(gameObject);
            source.PlayOneShot(thud, 0.7f);
            Player.score = Player.score + 1;
            
    }
}
------------------------------------------------------------------------------------------------------------------------

LoadLevel

using UnityEngine;
using System.Collections;

public class LoadLevel : MonoBehaviour {

    public void ChangeScene(string sceneDirection)
    {
        Application.LoadLevel(sceneDirection);
    }

}

------------------------------------------------------------------------------------------------------------------------
Player

using UnityEngine;
using System.Collections;

public class Player : MonoBehaviour {

	public float speed = 10f;
    public Vector2 maxVel = new Vector2(3, 5);
    public bool standing;
    public float jetSpeed = 15f;
    public float speedAirMulti = .8f;
    private PlayerController controller;
    private Animator animator;
    public static int score = 0 ; 

    void Start()
    {
        controller = GetComponent<PlayerController>();
        animator = GetComponent<Animator>();
    }

	void Update () {
        var forceX = 0f;
        var forceY = 0f;
        var absoluteVelX =  Mathf.Abs (GetComponent<Rigidbody2D>().velocity.x);
        var absoluteVelY = Mathf.Abs(GetComponent<Rigidbody2D>().velocity.y);


        if (absoluteVelY < .2f)
        {
            standing = true;
        }
        else
        {
            standing = false;
        }

        if (controller.moving.x != 0)
        {
            if (absoluteVelX < maxVel.x)
            {
                forceX = standing ? speed * controller.moving.x : (speed * controller.moving.x * speedAirMulti);
                transform.localScale = new Vector3(forceX > 0 ? 1 : -1, 1, 1);
            }
            animator.SetInteger("AnimState", 1);
            }
        else
        {
            animator.SetInteger("AnimState", 0);
        }

        if (controller.moving.y > 0)
        {
            if (absoluteVelY < maxVel.y)
            {
                forceY = jetSpeed * controller.moving.y;
                animator.SetInteger("AnimState", 2);
            }
        }
        GetComponent<Rigidbody2D>().AddForce(new Vector2(forceX, forceY));
	    
    }
}

------------------------------------------------------------------------------------------------------------------------

RandomSprite

using UnityEngine;
using System.Collections;


public class RandomSprite : MonoBehaviour {
    public Sprite[] sprites;
    public string resourceName;

	// Use this for initialization
	void Start () {
        if (resourceName != "")
            sprites = Resources.LoadAll<Sprite>(resourceName);
            GetComponent<SpriteRenderer>().sprite = sprites[Random.Range(0, sprites.Length)];
	
	}
	
	// Update is called once per frame
	void Update () {
	
	}
}

---------------------------------------------------------------------------------------------------------------------------------

ScoreManager

using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class ScoreManager : MonoBehaviour {

    public Text scoreText;
    public Text highScoreText;

    public float scoreC;
    public float highScoreC;
    public float pointsPSecond;

    public bool scoreUp;

    // Use this for initialization
	void Start () {
	
	}
	
	// Update is called once per frame
	void Update () {
        if (scoreUp) { 
        scoreC += pointsPSecond * Time.deltaTime;
        }
        if (scoreC > highScoreC)
        {
            highScoreC = scoreC;
        }
        scoreText.text = "Score: " + Mathf.Round(scoreC);
        highScoreText.text = "High Score: " + Mathf.Round(highScoreC);
	}
}

------------------------------------------------------------------------------------------------------------------------------
PlatformerCharachter2D (Editado significativamente)


using System;
using UnityEngine;

namespace UnityStandardAssets._2D
{
    public class PlatformerCharacter2D : MonoBehaviour
    {
        [SerializeField] private float m_MaxSpeed = 10f;                    // The fastest the player can travel in the x axis.
        [SerializeField] private float m_JumpForce = 400f;                  // Amount of force added when the player jumps.
        [Range(0, 1)] [SerializeField] private float m_CrouchSpeed = .36f;  // Amount of maxSpeed applied to crouching movement. 1 = 100%
        [SerializeField] private bool m_AirControl = false;                 // Whether or not a player can steer while jumping;
        [SerializeField] private LayerMask m_WhatIsGround;                  // A mask determining what is ground to the character

        private Transform m_GroundCheck;    // A position marking where to check if the player is grounded.
        const float k_GroundedRadius = .2f; // Radius of the overlap circle to determine if grounded
        private bool m_Grounded;            // Whether or not the player is grounded.
        private Transform m_CeilingCheck;   // A position marking where to check for ceilings
        const float k_CeilingRadius = .01f; // Radius of the overlap circle to determine if the player can stand up
        private Animator m_Anim;            // Reference to the player's animator component.
        private Rigidbody2D m_Rigidbody2D;
        private bool m_FacingRight = true;  // For determining which way the player is currently facing.

        bool m_DoubleJump = false;

        private void Awake()
        {
            // Setting up references.
            m_GroundCheck = transform.Find("GroundCheck");
            m_CeilingCheck = transform.Find("CeilingCheck");
            m_Anim = GetComponent<Animator>();
            m_Rigidbody2D = GetComponent<Rigidbody2D>();
        }


        private void FixedUpdate()
        {
            m_Grounded = false;

            // The player is grounded if a circlecast to the groundcheck position hits anything designated as ground
            // This can be done using layers instead but Sample Assets will not overwrite your project settings.
            Collider2D[] colliders = Physics2D.OverlapCircleAll(m_GroundCheck.position, k_GroundedRadius, m_WhatIsGround);
            for (int i = 0; i < colliders.Length; i++)
            {
                if (colliders[i].gameObject != gameObject)
                    m_Grounded = true;
            }
            m_Anim.SetBool("Ground", m_Grounded);

            // Set the vertical animation
            m_Anim.SetFloat("vSpeed", m_Rigidbody2D.velocity.y);
            if (m_Grounded)
                m_DoubleJump = false;


        }


        public void Move(float move, bool crouch, bool jump)
        {
            // If crouching, check to see if the character can stand up
            if (!crouch && m_Anim.GetBool("Crouch"))
            {
                // If the character has a ceiling preventing them from standing up, keep them crouching
                if (Physics2D.OverlapCircle(m_CeilingCheck.position, k_CeilingRadius, m_WhatIsGround))
                {
                    crouch = true;
                }
            }

            // Set whether or not the character is crouching in the animator
            m_Anim.SetBool("Crouch", crouch);

            //only control the player if grounded or airControl is turned on
            if (m_Grounded || m_AirControl)
            {
                // Reduce the speed if crouching by the crouchSpeed multiplier
                move = (crouch ? move*m_CrouchSpeed : move);

                // The Speed animator parameter is set to the absolute value of the horizontal input.
                m_Anim.SetFloat("Speed", Mathf.Abs(move));

                // Move the character
                m_Rigidbody2D.velocity = new Vector2(move*m_MaxSpeed, m_Rigidbody2D.velocity.y);

                // If the input is moving the player right and the player is facing left...
                if (move > 0 && !m_FacingRight)
                {
                    // ... flip the player.
                    Flip();
                }
                    // Otherwise if the input is moving the player left and the player is facing right...
                else if (move < 0 && m_FacingRight)
                {
                    // ... flip the player.
                    Flip();
                }
            }
            // If the player should jump...
            if ((m_Grounded || !m_DoubleJump) && jump && m_Anim.GetBool("Ground"))
            {
                // Add a vertical force to the player.
                m_Grounded = false;
                m_Anim.SetBool("Ground", false);
                m_Rigidbody2D.velocity = new Vector2(m_Rigidbody2D.velocity.x, 0);
                m_Rigidbody2D.AddForce(new Vector2(0f, m_JumpForce));
                if (!m_Grounded)
                    m_DoubleJump = true;
            }
        }


        private void Flip()
        {
            // Switch the way the player is labelled as facing.
            m_FacingRight = !m_FacingRight;

            // Multiply the player's x local scale by -1.
            Vector3 theScale = transform.localScale;
            theScale.x *= -1;
            transform.localScale = theScale;
        }
    }
}

---------------------------------------------------------------------------------------------------------------------------------

Platformer2DUserCotroler (Editado significativamente)

using System;
using UnityEngine;
using UnityStandardAssets.CrossPlatformInput;

namespace UnityStandardAssets._2D
{
    [RequireComponent(typeof (PlatformerCharacter2D))]
    public class Platformer2DUserControl : MonoBehaviour
    {
        private PlatformerCharacter2D m_Character;
        private bool m_Jump = false;


        private void Awake()
        {
            m_Character = GetComponent<PlatformerCharacter2D>();
        }


        private void Update()
        {
            if (!m_Jump)
            {
                // Read the jump input in Update so button presses aren't missed.
                m_Jump = CrossPlatformInputManager.GetButtonDown("Jump");
            }
        }


        private void FixedUpdate()
        {
            // Read the inputs.
            //bool crouch = Input.GetKey(KeyCode.LeftControl);
            //float h = CrossPlatformInputManager.GetAxis("Horizontal");
            // Pass all parameters to the character control script.
            m_Character.Move(1, false, m_Jump);
            m_Jump = false;
        }
    }
}

------------------------------------------------------------------------------------------------------------------------------

Jugador

using UnityEngine;
using System.Collections;

public class Player : MonoBehaviour {
    //Para la velocidad y salto
    public float velocidad;
    public float salto;

    //el player
    private Rigidbody2D cuerpoRigido;

    //piso
    public bool piso;
    public LayerMask quePiso;

    //colision
    private Collider2D colision;

    // Use this for initialization
    void Start () {

        cuerpoRigido = GetComponent<Rigidbody2D>();
        colision = GetComponent<Collider2D>();
    }
    
    // Update is called once per frame
    void Update () {

        piso = Physics2D.IsTouchingLayers (colision, quePiso);

        cuerpoRigido.velocity = new Vector2 (velocidad, cuerpoRigido.velocity.y); //se mueva de manera continua

        if(Input.GetKeyDown(KeyCode.Space) || Input.GetMouseButtonDown(0))//cuando toque la tecla espacio o el mouse
        {
            if(piso)
            {
                cuerpoRigido.velocity = new Vector2(cuerpoRigido.velocity.x, salto);
            }
        }

    }
}

------------------------------------------------------------------------------------------------------------------------------

Plataforma(generador)


using UnityEngine;
using System.Collections;

public class PlatGen : MonoBehaviour {
    public GameObject laPlat;
    public Transform coordenada;
    public float disInter;

    private float platAcho;

    public float platAnchoMin;
    public float platAnchoMax;

    //public GameObject[] theArray;
    private int formSelector;
    private float [] platFormsW;

    public objectcr [] theObj2;

    //
    private float altMin;
    public Transform maxAltCoor;
    private float altMax;
    public float maxAltCam;
    private float alturaCam;


    // Use this for initialization
    void Start () {
        platFormsW = new float[theObj2.Length];

        for (int i= 0; i<theObj2.Length; i++) {
        
            platFormsW[i]=theObj2[i].pooObj.GetComponent<BoxCollider2D>().size.x;
        }
        altMin = transform.position.y;
        altMax = maxAltCoor.position.y;
    }
    
    // Update is called once per frame
    void Update () {
        if (transform.position.x < coordenada.position.x) 
        {
            disInter=Random.Range(platAnchoMin, platAnchoMax);

            formSelector=Random.Range(0, theObj2.Length);

            alturaCam= transform.position.y+Random.Range(maxAltCam,-maxAltCam);

            if (alturaCam>altMax){

                alturaCam=altMax;
            
            }else if (alturaCam<altMin){

                alturaCam=altMin;
            }


            transform.position = new Vector3(transform.position.x + (platFormsW[formSelector]/2)+ disInter, alturaCam, transform.position.z);

            //Instantiate (/*laPlat*/ theObj2[formSelector], transform.position, transform.rotation);

            GameObject newPlat=theObj2[formSelector].getPooledObject();

            newPlat.transform.position=transform.position;
            newPlat.transform.rotation=transform.rotation;
            newPlat.SetActive(true);

            transform.position = new Vector3(transform.position.x + (platFormsW[formSelector]/2),transform.position.y, transform.position.z);

        }

    }
}
---------------------------------------------------------------------------------------------------------------------------------

Camara movimiento

using UnityEngine;
using System.Collections;

public class CamCon : MonoBehaviour {
    public Player jugador;

    private Vector3 ultimaPosicion;
    private float mDistancia;


    // Use this for initialization
    void Start () {
        jugador = FindObjectOfType<Player> ();
        ultimaPosicion = jugador.transform.position;
    }
    
    // Update is called once per frame
    void Update () {
        mDistancia = jugador.transform.position.x - ultimaPosicion.x;
        transform.position = new Vector3 (transform.position.x + mDistancia, transform.position.y, transform.position.z);

        ultimaPosicion = jugador.transform.position;
    }
}
------------------------------------------------------------------------------------------------------------------------------

Destructor de plataforma

using UnityEngine;
using System.Collections;

public class PlatDes : MonoBehaviour {

    public GameObject platDesCoor;

    // Use this for initialization
    void Start () {
        platDesCoor = GameObject.Find ("PlatDesP");
    }
    
    // Update is called once per frame
    void Update () {

        if (transform.position.x<platDesCoor.transform.position.x)
        {
            //Destroy (gameObject);
            gameObject.SetActive(false);
        }
    }
}

------------------------------------------------------------------------------------------------------------------------------

Object

using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class objectcr : MonoBehaviour {
    public GameObject pooObj;
    public int poolAm;

    List<GameObject> poolObjcts;
    // Use this for initialization
    void Start () {

        poolObjcts = new List<GameObject> ();
        for (int i=0; i<poolAm; i++) {
            GameObject obj = (GameObject)Instantiate (pooObj);
            obj.SetActive (false);
            poolObjcts.Add (obj);
        }
    }
    public GameObject getPooledObject(){
        for (int i=0; i<poolObjcts.Count; i++) {
            if (!poolObjcts[i].activeInHierarchy){
                return poolObjcts[i];
            }
        }
        GameObject obj = (GameObject)Instantiate (pooObj);
        obj.SetActive (false);
        poolObjcts.Add (obj);
        return obj;
    }
}
