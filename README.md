using System;
using System.Drawing;
using System.Windows.Forms;

public class BrickBreakerGame : Form
{
    private Timer gameTimer;
    private int ballXSpeed = 5;
    private int ballYSpeed = -5;
    private int paddleSpeed = 15;
    private Rectangle ball;
    private Rectangle paddle;
    private Rectangle[] bricks;
    private int brickRows = 5;
    private int brickCols = 10;
    private int brickWidth = 75;
    private int brickHeight = 30;

    public BrickBreakerGame()
    {
        this.DoubleBuffered = true; // Reduce flickering
        this.Width = 800;
        this.Height = 600;

        ball = new Rectangle(400, 300, 20, 20);
        paddle = new Rectangle(350, 550, 100, 20);
        InitializeBricks();

        gameTimer = new Timer();
        gameTimer.Interval = 20;
        gameTimer.Tick += GameLoop;
        gameTimer.Start();

        this.Paint += new PaintEventHandler(DrawGame);
        this.KeyDown += new KeyEventHandler(HandleKeyDown);
        this.KeyUp += new KeyEventHandler(HandleKeyUp);
    }

    private void InitializeBricks()
    {
        bricks = new Rectangle[brickRows * brickCols];
        for (int row = 0; row < brickRows; row++)
        {
            for (int col = 0; col < brickCols; col++)
            {
                int x = col * brickWidth + 10;
                int y = row * brickHeight + 10;
                bricks[row * brickCols + col] = new Rectangle(x, y, brickWidth - 5, brickHeight - 5);
            }
        }
    }

    private void GameLoop(object sender, EventArgs e)
    {
        MoveBall();
        CheckCollisions();
        this.Invalidate(); // Trigger a redraw
    }

    private void MoveBall()
    {
        ball.X += ballXSpeed;
        ball.Y += ballYSpeed;

        if (ball.X < 0 || ball.X > this.Width - ball.Width)
            ballXSpeed = -ballXSpeed;

        if (ball.Y < 0)
            ballYSpeed = -ballYSpeed;

        if (ball.Y > this.Height)
        {
            gameTimer.Stop();
            MessageBox.Show("Game Over!");
            Application.Exit();
        }
    }

    private void CheckCollisions()
    {
        // Check if ball hits the paddle
        if (ball.IntersectsWith(paddle))
        {
            ballYSpeed = -ballYSpeed;
            ball.Y = paddle.Top - ball.Height; // Move the ball above the paddle
        }

        // Check if ball hits the bricks
        for (int i = 0; i < bricks.Length; i++)
        {
            if (bricks[i] != Rectangle.Empty && ball.IntersectsWith(bricks[i]))
            {
                bricks[i] = Rectangle.Empty; // Remove the brick
                ballYSpeed = -ballYSpeed; // Reverse ball direction
                break; // Only break one brick at a time
            }
        }
    }

    private void DrawGame(object sender, PaintEventArgs e)
    {
        e.Graphics.Clear(Color.Black);
        e.Graphics.FillEllipse(Brushes.Red, ball);
        e.Graphics.FillRectangle(Brushes.Blue, paddle);

        foreach (var brick in bricks)
        {
            if (brick != Rectangle.Empty)
                e.Graphics.FillRectangle(Brushes.Yellow, brick);
        }
    }

    private void HandleKeyDown(object sender, KeyEventArgs e)
    {
        if (e.KeyCode == Keys.Left && paddle.Left > 0)
            paddle.X -= paddleSpeed;

        if (e.KeyCode == Keys.Right && paddle.Right < this.Width)
            paddle.X += paddleSpeed;
    }

    private void HandleKeyUp(object sender, KeyEventArgs e)
    {
        // You can handle key release if needed
    }

    [STAThread]
    public static void Main()
    {
        Application.EnableVisualStyles();
        Application.Run(new BrickBreakerGame());
    }
}
