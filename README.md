# iosgame

Created a version of flappy bird using swift to code it in xcode using jpegs for the bird and pipe images. I also added a restart feature once a game ends and a score tracker to update the score once you go through a pipe.

<img width="472" alt="Screenshot 2025-03-20 at 11 48 21 PM" src="https://github.com/user-attachments/assets/c618076e-a0f5-4351-97ff-a931d8689abb" />

//
//  GameScene.swift
//  Flappy
//
//  Created by Adam Klein on 3/5/25.
//

import SpriteKit

class GameScene: SKScene, SKPhysicsContactDelegate {
    
    var bird = SKSpriteNode()
    var gameStarted = false
    var score = 0
    var scoreLabel = SKLabelNode()
    var gameOver = false
    
    override func didMove(to view: SKView) {
        self.physicsWorld.contactDelegate = self
        self.backgroundColor = SKColor.cyan
        
        gameOver = false
        gameStarted = false
        score = 0
        
        setupBird()
        setupGround()
        setupScoreLabel()
    }
    
    func startGame() {
        let spawnPipes = SKAction.run { self.createPipes() }
        let delay = SKAction.wait(forDuration: 2.0)
        let spawnSequence = SKAction.sequence([spawnPipes, delay])
        let spawnForever = SKAction.repeatForever(spawnSequence)
        
        self.run(spawnForever)
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        if gameOver {
            restartGame()
        } else if !gameStarted {
            gameStarted = true
            bird.physicsBody?.affectedByGravity = true
            startGame()
        }
        
        bird.physicsBody?.velocity = CGVector(dx: 0, dy: 0)
        bird.physicsBody?.applyImpulse(CGVector(dx: 0, dy: 30))
    }
    
    func didBegin(_ contact: SKPhysicsContact) {
        let firstBody = contact.bodyA
        let secondBody = contact.bodyB
        
        if (firstBody.categoryBitMask == 1 && secondBody.categoryBitMask == 4) ||
            (firstBody.categoryBitMask == 4 && secondBody.categoryBitMask == 1) {
            score += 1
            scoreLabel.text = "Score: \(score)"
        } else if (firstBody.categoryBitMask == 1 && secondBody.categoryBitMask == 2) ||
                    (firstBody.categoryBitMask == 2 && secondBody.categoryBitMask == 1) {
            gameOver = true
            self.removeAllActions()
            self.speed = 0
            bird.physicsBody?.velocity = CGVector(dx: 0, dy: 0)
        }
    }
    
    func createPipes() {
        let pipeUpTexture = SKTexture(imageNamed: "pipeDown")
        let pipeDownTexture = SKTexture(imageNamed: "pipeUp")
        
        let pipeWidth: CGFloat = 80
        let pipeHeight: CGFloat = 400
        
        let pipeUp = SKSpriteNode(texture: pipeUpTexture)
        let pipeDown = SKSpriteNode(texture: pipeDownTexture)
        
        pipeUp.size = CGSize(width: pipeWidth, height: pipeHeight)
        pipeDown.size = CGSize(width: pipeWidth, height: pipeHeight)
        
        let gapHeight = bird.size.height * 12
        let movementAmount = CGFloat(arc4random_uniform(UInt32(self.frame.height / 4)))
        let pipeOffset = movementAmount - self.frame.height / 4
        
        pipeUp.position = CGPoint(x: self.frame.width, y: self.frame.midY + pipeOffset + gapHeight / 2)
        pipeDown.position = CGPoint(x: self.frame.width, y: self.frame.midY + pipeOffset - gapHeight / 2)
        
        for pipe in [pipeUp, pipeDown] {
            pipe.physicsBody = SKPhysicsBody(rectangleOf: pipe.size)
            pipe.physicsBody?.isDynamic = false
            pipe.physicsBody?.categoryBitMask = 2
            self.addChild(pipe)
        }
        
        let scoreNode = SKNode()
        scoreNode.position = CGPoint(x: pipeUp.position.x + pipeUp.size.width / 2, y: self.frame.midY + pipeOffset)
        scoreNode.physicsBody = SKPhysicsBody(rectangleOf: CGSize(width: 1, height: gapHeight))
        scoreNode.physicsBody?.isDynamic = false
        scoreNode.physicsBody?.categoryBitMask = 4
        scoreNode.physicsBody?.contactTestBitMask = 1
        self.addChild(scoreNode)
        
        let movePipes = SKAction.moveBy(x: -self.frame.width - 100, y: 0, duration: 4)
        let removePipes = SKAction.removeFromParent()
        let pipeSequence = SKAction.sequence([movePipes, removePipes])
        
        pipeUp.run(pipeSequence)
        pipeDown.run(pipeSequence)
        scoreNode.run(pipeSequence)
    }
    
    func setupBird() {
        bird = SKSpriteNode(imageNamed: "bird")
        bird.position = CGPoint(x: self.frame.midX, y: self.frame.midY)
        bird.size = CGSize(width: 50, height: 50)
        
        bird.physicsBody = SKPhysicsBody(circleOfRadius: bird.size.width / 2)
        bird.physicsBody?.isDynamic = true
        bird.physicsBody?.affectedByGravity = false
        bird.physicsBody?.categoryBitMask = 1
        bird.physicsBody?.collisionBitMask = 2
        bird.physicsBody?.contactTestBitMask = 2 | 4
        
        self.addChild(bird)
    }
    
    func setupGround() {
        let ground = SKSpriteNode(color: .brown, size: CGSize(width: self.frame.width, height: 50))
        ground.position = CGPoint(x: self.frame.midX, y: self.frame.minY + 25)
        
        ground.physicsBody = SKPhysicsBody(rectangleOf: ground.size)
        ground.physicsBody?.isDynamic = false
        ground.physicsBody?.categoryBitMask = 2
        
        self.addChild(ground)
    }
    
    func setupScoreLabel() {
        scoreLabel = SKLabelNode(fontNamed: "Arial")
        scoreLabel.text = "Score: 0"
        scoreLabel.fontSize = 60
        scoreLabel.fontColor = .black
        scoreLabel.position = CGPoint(x: self.frame.midX, y: self.frame.maxY - 100)
        
        self.addChild(scoreLabel)
    }
    
    func restartGame() {
        self.removeAllChildren()
        
        gameOver = false
        gameStarted = false
        score = 0
        self.speed = 1
        
        didMove(to: self.view!)
    }
}





