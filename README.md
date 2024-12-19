# Snakegame-using-Java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.util.ArrayList;
import java.util.Random;

public class SnakeGame extends JPanel implements ActionListener {
    private final int TILE_SIZE = 30;
    private final int WIDTH = 600;
    private final int HEIGHT = 400;
    private final int ALL_TILES = (WIDTH * HEIGHT) / (TILE_SIZE * TILE_SIZE);
    private final int DELAY = 100;

    private ArrayList<Point> snake;
    private Point food;
    private int direction;
    private boolean running;
    private Timer timer;

    private static final int UP = 0;
    private static final int RIGHT = 1;
    private static final int DOWN = 2;
    private static final int LEFT = 3;

    public SnakeGame() {
        setPreferredSize(new Dimension(WIDTH, HEIGHT));
        setBackground(Color.BLACK);
        setFocusable(true);

        snake = new ArrayList<>();
        snake.add(new Point(5 * TILE_SIZE, 5 * TILE_SIZE));
        direction = RIGHT;
        running = true;

        timer = new Timer(DELAY, this);
        timer.start();

        addKeyListener(new KeyAdapter() {
            @Override
            public void keyPressed(KeyEvent e) {
                int key = e.getKeyCode();
                if (key == KeyEvent.VK_UP && direction != DOWN) direction = UP;
                if (key == KeyEvent.VK_RIGHT && direction != LEFT) direction = RIGHT;
                if (key == KeyEvent.VK_DOWN && direction != UP) direction = DOWN;
                if (key == KeyEvent.VK_LEFT && direction != RIGHT) direction = LEFT;
            }
        });

        spawnFood();
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        if (running) {
            g.setColor(Color.RED);
            g.fillRect(food.x, food.y, TILE_SIZE, TILE_SIZE);

            g.setColor(Color.GREEN);
            for (Point point : snake) {
                g.fillRect(point.x, point.y, TILE_SIZE, TILE_SIZE);
            }
        } else {
            showGameOver(g);
        }
    }

    private void move() {
        Point head = snake.get(0);
        Point newHead = new Point(head.x, head.y);

        switch (direction) {
            case UP:
                newHead.translate(0, -TILE_SIZE);
                break;
            case RIGHT:
                newHead.translate(TILE_SIZE, 0);
                break;
            case DOWN:
                newHead.translate(0, TILE_SIZE);
                break;
            case LEFT:
                newHead.translate(-TILE_SIZE, 0);
                break;
        }

        if (newHead.equals(food)) {
            snake.add(0, food);
            spawnFood();
        } else {
            snake.add(0, newHead);
            snake.remove(snake.size() - 1);
        }

        checkCollisions();
    }

    private void checkCollisions() {
        Point head = snake.get(0);

        if (head.x < 0 || head.x >= WIDTH || head.y < 0 || head.y >= HEIGHT) {
            running = false;
        }

        for (int i = 1; i < snake.size(); i++) {
            if (head.equals(snake.get(i))) {
                running = false;
            }
        }

        if (!running) {
            timer.stop();
        }
    }

    private void spawnFood() {
        Random random = new Random();
        int x = random.nextInt(WIDTH / TILE_SIZE) * TILE_SIZE;
        int y = random.nextInt(HEIGHT / TILE_SIZE) * TILE_SIZE;
        food = new Point(x, y);

        while (snake.contains(food)) {
            x = random.nextInt(WIDTH / TILE_SIZE) * TILE_SIZE;
            y = random.nextInt(HEIGHT / TILE_SIZE) * TILE_SIZE;
            food = new Point(x, y);
        }
    }

    private void showGameOver(Graphics g) {
        String msg = "Game Over";
        Font font = new Font("Helvetica", Font.BOLD, 14);
        FontMetrics metrics = getFontMetrics(font);

        g.setColor(Color.WHITE);
        g.setFont(font);
        g.drawString(msg, (WIDTH - metrics.stringWidth(msg)) / 2, HEIGHT / 2);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (running) {
            move();
        }
        repaint();
    }

    public static void main(String[] args) {
        JFrame frame = new JFrame("Snake Game");
        SnakeGame game = new SnakeGame();

        frame.add(game);
        frame.pack();
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setVisible(true);
    }
}
