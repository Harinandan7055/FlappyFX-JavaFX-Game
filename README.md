/**
*  FlappyFX
*  A game in JavaFX
*/

package flappybird;

import javafx.application.*;
import javafx.event.*;
import javafx.scene.*;
import javafx.scene.control.*;
import javafx.scene.layout.*;
import javafx.stage.*;
import javafx.scene.image.*;
import javafx.animation.*;
import javafx.geometry.*;
import javafx.scene.input.*;
import javafx.util.*;
import java.util.*;
import java.io.*;
import javafx.beans.property.*;
import javafx.scene.effect.*;
import javafx.scene.paint.*;
import javafx.scene.text.*;
import javafx.scene.transform.*;
import javafx.scene.media.*;
import javafx.scene.shape.*;
import javafx.scene.web.*;

public class FlappyBird extends Application
{
    private Label flappyLabel, backgroundLabel, scoreLabel, pillerLabel [], gameOverlabel, screenLabel, highestScoreLabel;
    private Timeline flappyJumpTimeline, gravityTimeline, backgroundTimeline, pillerTimeline[];
    private RotateTransition rotateTransition;
    private Stage stage;
    private Rectangle2D screenDimension;
    private double screenWidth, screenHeight, spacingHeight, flappyBirdJump;
    private Random random;
    private Pane root,settingPane, gamePane, gameOverPane;
    private Button startPauseButton, exitButton, settingButton, helpButton;
    private Slider lifeSlider, volumeControlSlider, pillerSpacingSlider, flappyJumpHeightSlider;
    private boolean gravityFlag, pillerStrike[];
    private IntegerProperty score, highestScore;
    private FadeTransition fadeTransition;
    private Ellipse flappyBoundCircle;
    private ProgressBar lifeBar;
    private EventHandler<KeyEvent> eventHandler;
    private Timeline gameTimeline;
    
    @Override
    public void init()
    {
        screenDimension = Screen.getPrimary().getBounds();
        screenWidth = screenDimension.getWidth();
        screenHeight = screenDimension.getHeight();
        random = new Random();
    }
    
    
    
    @Override
    public void start(Stage primaryStage)
    {
        stage = primaryStage;
        root = new Pane();
        root.setId("root");
        Scene scene = new Scene(root, screenWidth, screenHeight, Color.GREEN);
        scene.getStylesheets().add(getClass().getResource("/resourcepack/style.css").toExternalForm());
        primaryStage.initStyle(StageStyle.UNDECORATED);
        primaryStage.setTitle("FlappyFX");
        primaryStage.getIcons().add(new Image(getClass().getResourceAsStream("/resourcepack/FlappyBird2.png")));
        primaryStage.setScene(scene);
        primaryStage.setFullScreen(true);
        primaryStage.setFullScreenExitHint("");
        KeyCombination keyCombination = new KeyCodeCombination(KeyCode.F4, KeyCodeCombination.ALT_DOWN);
        primaryStage.setFullScreenExitKeyCombination(keyCombination);
        primaryStage.setAlwaysOnTop(true);
        primaryStage.show();
        flappyFishStartAnimation();
    }
    
    public void createHelpAndSupport()
    {
        ImageView imageView = new ImageView(new Image(this.getClass().getResourceAsStream("/resourcepack/me.jpg")));
        imageView.setFitWidth(250);
        imageView.setFitHeight(400);
        
        String gradientString = "from 0% 0% to 100% 0%, reflect, green 0%, yellow 25%, green 50%, yellow 75%, green 100%";
        Text gameText = new Text("FlappyFX");
        gameText.setTextAlignment(TextAlignment.CENTER);
        gameText.setTextOrigin(VPos.TOP);
        gameText.setFont(Font.font(50));
        gameText.setUnderline(true);
        LinearGradient gradient = LinearGradient.valueOf(gradientString);
        gameText.setFill(gradient);
        
        ImageView gameIcon = new ImageView(new Image(this.getClass().getResourceAsStream("/resourcepack/FlappyBird2.png")));
        gameIcon.setFitWidth(50);
        gameIcon.setFitHeight(50);
        
        HBox hBox1 = new HBox(gameIcon, gameText);
        hBox1.setAlignment(Pos.CENTER);
        
        Text helpTex = new Text("Help Content");
        helpTex.setTextAlignment(TextAlignment.CENTER);
        helpTex.setTextOrigin(VPos.TOP);
        helpTex.setFont(Font.font(20));
        helpTex.setUnderline(true);
        helpTex.setFill(gradient);
        
        String helpStr = "";
        File helpFile = new File("Help.bin");
        if(!helpFile.exists())
        {
            System.out.println("Help File is Currupted or Deleted");
            return;
        }
        try(RandomAccessFile accessFile = new RandomAccessFile(helpFile, "r"))
        {
            String temp = "";
            while( (temp= accessFile.readLine()) != null)
            {
                helpStr+=temp+"\n";
            }
        }
        catch(IOException e)
        {
            System.out.println("IOException: "+e.getMessage());
        }
        
        Text helpText = new Text(helpStr);
        ScrollPane scrollPane = new ScrollPane(helpText);
        scrollPane.setPrefHeight(160);
        
        Text devText = new Text("Developed By: ");
        devText.setTextAlignment(TextAlignment.CENTER);
        devText.setTextOrigin(VPos.TOP);
        devText.setFont(Font.font("arial", FontWeight.BOLD, 10));
        devText.setFill(gradient);
        
        Text nameText = new Text("Harinandan Kumar");
        nameText.setTextAlignment(TextAlignment.CENTER);
        nameText.setTextOrigin(VPos.TOP);
        nameText.setFont(Font.font(30));
        nameText.setFill(gradient);
        
        HBox hBox2 = new HBox(devText, nameText);
        hBox2.setAlignment(Pos.CENTER);
        
        Text emText = new Text("Email: ");
        emText.setTextAlignment(TextAlignment.CENTER);
        emText.setTextOrigin(VPos.TOP);
        emText.setFont(Font.font("arial", FontWeight.BOLD, 10));
        emText.setFill(gradient);
        
        Hyperlink mailText = new Hyperlink("Harinandan_EHV@live.in");
        mailText.setTextAlignment(TextAlignment.CENTER);
        mailText.setFont(Font.font(25));
        mailText.setTextFill(gradient);
        
        HBox hBox3 = new HBox(emText, mailText);
        hBox3.setAlignment(Pos.CENTER);
        
        VBox vBox = new VBox(5, hBox1,new Separator(), helpTex, scrollPane, new Separator(), hBox2, hBox3);
        vBox.setStyle("-fx-background-image: url(/resourcepack/Gridimage.png);" +
                    "-fx-background-size: 350px 400px;" +
                    "-fx-background-position: right;" +
                    "-fx-background-color: transparent;" +
                    "-fx-background-repeat: repeat;" +
                    "-fx-background-radius: 50px;");
        vBox.setAlignment(Pos.CENTER);
        vBox.setPrefSize(350, 400);
        vBox.setTranslateX(250);
        
        Pane root = new Pane(imageView, vBox);
        Scene scene = new Scene(root, 600, 400, Color.BLUE);
        Stage myStage = new Stage(StageStyle.UTILITY);
        myStage.setScene(scene);
        myStage.initModality(Modality.APPLICATION_MODAL);
        myStage.setResizable(false);
        myStage.setTitle("Help and Support");
        mailText.setOnAction((ae) ->
        {
            root.getChildren().clear();
            WebView webView = new WebView();
            ProgressIndicator indicator = new ProgressIndicator(0);
            indicator.setPrefSize(100, 100);
            indicator.setTranslateX((myStage.getWidth()-indicator.getPrefWidth())/2);
            indicator.setTranslateY((myStage.getHeight()-indicator.getPrefHeight())/2);
            webView.setPrefSize(myStage.getWidth()-5, myStage.getHeight()-5);
            root.getChildren().addAll(webView, indicator);
            WebEngine webEngine = webView.getEngine();
            indicator.progressProperty().bind(webEngine.getLoadWorker().progressProperty());
            webEngine.load("https://www.facebook.com/profile.php?id=100008951404388");
        });
        myStage.setAlwaysOnTop(true);
        myStage.showAndWait();
    }
    
    public void setPillerPosition(int index)
    {
        int y = random.nextInt(400);
        while(y < pillerSpacingSlider.getValue())
        {
            y = random.nextInt(400);
        }
        pillerLabel[index].setTranslateY(-y);
        pillerLabel[index+1].setTranslateY(-y+spacingHeight+600);
    }
    
    
    public void jumpAnimation()
    {
        if(gravityTimeline != null)
        {
            gravityTimeline.stop();
        }
        if(flappyJumpTimeline != null)
        {
            flappyJumpTimeline.stop();
        }
        KeyValue kv1 = new KeyValue(flappyLabel.translateYProperty(), flappyLabel.getTranslateY(), Interpolator.LINEAR);
        KeyFrame kf1 = new KeyFrame(Duration.ZERO, kv1);
        KeyValue kvr1 = new KeyValue(flappyLabel.rotateProperty(), 0, Interpolator.LINEAR);
        KeyFrame kfr1 = new KeyFrame(Duration.ZERO, kvr1);
        KeyValue kv2 = new KeyValue(flappyLabel.translateYProperty(), flappyLabel.getTranslateY()-flappyBirdJump, Interpolator.LINEAR);
        KeyFrame kf2 = new KeyFrame(Duration.millis(500), kv2);
        KeyValue kvr2 = new KeyValue(flappyLabel.rotateProperty(), -60, Interpolator.LINEAR);
        KeyFrame kfr2 = new KeyFrame(Duration.millis(500), kvr2);
        KeyValue kv3 = new KeyValue(flappyLabel.translateYProperty(), flappyLabel.getTranslateY()-flappyBirdJump, Interpolator.LINEAR);
        KeyFrame kf3 = new KeyFrame(Duration.millis(600), kv2);
        
        flappyJumpTimeline = new Timeline(kf1, kfr1, kf2, kfr2);
        flappyJumpTimeline.setOnFinished((ae) -> 
        {
            gravityAnimation();
            gravityFlag = true;
        });
        flappyJumpTimeline.play();
    }
    
    public void gravityAnimation()
    {
        if(gravityTimeline != null)
        {
            gravityTimeline.stop();
        }
        if(flappyJumpTimeline != null)
        {
            flappyJumpTimeline.stop();
        }
        KeyValue kv1 = new KeyValue(flappyLabel.translateYProperty(), flappyLabel.getTranslateY(), Interpolator.EASE_IN);
        KeyFrame kf1 = new KeyFrame(Duration.ZERO, kv1);
        KeyValue kvr1 = new KeyValue(flappyLabel.rotateProperty(), -60, Interpolator.EASE_IN);
        KeyFrame kfr1 = new KeyFrame(Duration.ZERO, kvr1);
        KeyValue kv2 = new KeyValue(flappyLabel.translateYProperty(), stage.getHeight()-flappyLabel.getHeight(), Interpolator.LINEAR);
        KeyFrame kf2 = new KeyFrame(Duration.millis(1000), kv2);
        KeyValue kvr2 = new KeyValue(flappyLabel.rotateProperty(), 60, Interpolator.LINEAR);
        KeyFrame kfr2 = new KeyFrame(Duration.millis(1000), kvr2);
        
        gravityTimeline = new Timeline(kf1, kfr1, kf2, kfr2);
        gravityTimeline.play();
    }
    
    int aa=0;
    public void pillerAnimation(int index, int delayInMillies)
    {
        KeyValue kv1 = new KeyValue(pillerLabel[index].translateXProperty(), screenWidth+60, Interpolator.LINEAR);
        KeyFrame kf1 = new KeyFrame(Duration.ZERO, this::resetPillerStrike, kv1);
        KeyValue kv2 = new KeyValue(pillerLabel[index].translateXProperty(), -100, Interpolator.LINEAR);
        KeyFrame kf2 = new KeyFrame(Duration.seconds(5),(ae)-> setPillerPosition(index), kv2);
        
        pillerTimeline[aa] = new Timeline(kf1, kf2);
        pillerTimeline[aa].setCycleCount(Timeline.INDEFINITE);
        pillerTimeline[aa].setDelay(Duration.millis(delayInMillies));
        pillerTimeline[aa].setRate(1);
        pillerTimeline[aa].play();
        aa++;
    }
    
    int pillerIndex=0;
    public void resetPillerStrike(ActionEvent ae)
    {
        pillerStrike[pillerIndex++] = true;
        if(pillerIndex >= 4)
        {
            pillerIndex = 0;
        }   
    }
    
    public void createSettingPane()
    {
        settingPane = new Pane();
        settingPane.setId("settingPane");
        settingPane.setPrefSize(screenWidth, screenHeight);
        settingPane.setStyle("-fx-background-color: grey;");
        aa=0;
        startPauseButton = new Button("Start");
        settingButton = new Button("Settings");
        exitButton = new Button("Exit");
        helpButton = new Button("Help");
        startPauseButton.setPrefSize(400, 100);
        settingButton.setPrefSize(400, 100);
        exitButton.setPrefSize(400, 100);
        helpButton.setPrefSize(400, 100);
        VBox buttonBox = new VBox(10,startPauseButton,settingButton, helpButton,exitButton);
        Label lifeLabel = new Label("Life");
        lifeLabel.setAlignment(Pos.CENTER);
        lifeLabel.setPrefWidth(400);
        lifeSlider = new Slider(1, 10, 3);
        lifeSlider.setPrefWidth(400);
        lifeSlider.setMajorTickUnit(1);
        lifeSlider.setShowTickMarks(true);
        lifeSlider.setSnapToTicks(true);
        lifeSlider.setShowTickLabels(true);
        lifeSlider.setMinorTickCount(0);
        lifeSlider.valueProperty().addListener((pc, ov, nv) ->
        {
            lifeLabel.setText("Life: Min(1), Max(10), Current Value("+nv.intValue()+")");
        });
        Label volumeLabel = new Label("Volume Control");
        volumeLabel.setAlignment(Pos.CENTER);
        volumeLabel.setPrefWidth(400);
        volumeControlSlider = new Slider(0, 1, 1);
        volumeControlSlider.setPrefWidth(400);
        volumeControlSlider.setMajorTickUnit(0.1);
        volumeControlSlider.setShowTickMarks(true);
        volumeControlSlider.setSnapToTicks(true);
        volumeControlSlider.setShowTickLabels(true);
        volumeControlSlider.setMinorTickCount(4);
        volumeControlSlider.valueProperty().addListener((pc, ov, nv) ->
        {
            Formatter formatter = new Formatter();
            formatter.format("%.2f", nv.doubleValue());
            volumeLabel.setText("Volume Control: Min(0), Max(1), Current Value("+formatter+")");
        });
        Label pillerSpacingLabel = new Label("Piller Spacing");
        pillerSpacingLabel.setAlignment(Pos.CENTER);
        pillerSpacingLabel.setPrefWidth(400);
        pillerSpacingSlider = new Slider(50, 300, 180);
        pillerSpacingSlider.setPrefWidth(400);
        pillerSpacingSlider.setMajorTickUnit(50);
        pillerSpacingSlider.setShowTickMarks(true);
        pillerSpacingSlider.setSnapToTicks(true);
        pillerSpacingSlider.setShowTickLabels(true);
        pillerSpacingSlider.setMinorTickCount(4);
        pillerSpacingSlider.valueProperty().addListener((pc, ov, nv) ->
        {
            pillerSpacingLabel.setText("Piller Spacing: Min(50), Max(300), Current Value("+nv.intValue()+")");
            spacingHeight = nv.doubleValue();
        });
        Label flappyJumpLabel = new Label("Jump Height");
        flappyJumpLabel.setAlignment(Pos.CENTER);
        flappyJumpLabel.setPrefWidth(400);
        flappyJumpHeightSlider = new Slider(100, 500, 150);
        flappyJumpHeightSlider.setPrefWidth(400);
        flappyJumpHeightSlider.setMajorTickUnit(50);
        flappyJumpHeightSlider.setShowTickMarks(true);
        flappyJumpHeightSlider.setSnapToTicks(true);
        flappyJumpHeightSlider.setShowTickLabels(true);
        flappyJumpHeightSlider.setMinorTickCount(4);
        flappyJumpHeightSlider.valueProperty().addListener((pc, ov, nv) ->
        {
            flappyJumpLabel.setText("Jump Height: Min(100), Max(500), Current Value("+nv.intValue()+")");
            flappyBirdJump = nv.doubleValue();
        });
        VBox controlBox = new VBox(lifeLabel, lifeSlider, volumeLabel,volumeControlSlider,pillerSpacingLabel,pillerSpacingSlider,flappyJumpLabel,flappyJumpHeightSlider);
        lifeLabel.getStyleClass().add("settingLabel");
        volumeLabel.getStyleClass().add("settingLabel");
        pillerSpacingLabel.getStyleClass().add("settingLabel");
        flappyJumpLabel.getStyleClass().add("settingLabel");
        buttonBox.setPrefSize(400, 430);
        buttonBox.relocate(200, (screenHeight-buttonBox.getPrefHeight())/2);
        controlBox.relocate(buttonBox.getLayoutX()+buttonBox.getPrefWidth()+50, screenHeight/2-100);
        controlBox.getStyleClass().add("layouts");
        startPauseButton.setOnAction((ae) ->
        {
            if(startPauseButton.getText().equalsIgnoreCase("start"))
            {
                initSetup();
                startPauseButton.setText("Stop");
                gamePane = new Pane();
                gamePane.getChildren().add(backgroundLabel);
                gamePane.getChildren().addAll(pillerLabel);
                gamePane.getChildren().addAll(scoreLabel, lifeBar, flappyLabel, flappyBoundCircle);
                root.getChildren().add(gamePane);
                settingPane.setVisible(false);
                settingPane.toBack();
            }
            else if(startPauseButton.getText().equalsIgnoreCase("resume"))
            {
                resumeGame();
                settingPane.setVisible(false);
                settingPane.toBack();
                startPauseButton.setText("Stop");
            }
            stage.getScene().addEventFilter(KeyEvent.KEY_PRESSED, eventHandler);
        });
        settingButton.setOnAction((ae) ->
        {
            if(controlBox.isVisible())
            {
                controlBox.setVisible(false);
            }
            else
            {
                controlBox.setVisible(true);
            }
        });
        helpButton.setOnAction((ae) -> createHelpAndSupport());
        exitButton.setOnAction((ae) -> System.exit(0));
        settingPane.getChildren().addAll(buttonBox, controlBox);
        controlBox.setVisible(false);
        root.getChildren().add(settingPane);
    }
    
    public void pauseGame()
    {
        if(flappyJumpTimeline != null && !gravityFlag)
        {
            flappyJumpTimeline.pause();
        }
        if(gravityTimeline != null && gravityFlag)
        {
            gravityTimeline.pause();
        }
        if(backgroundTimeline != null)
        {
            backgroundTimeline.pause();
        }
        if(pillerTimeline[0] != null)
        {
            for(int i=0; i<4; i++)
            {
                pillerTimeline[i].pause();
            }
        }
        if(gameTimeline != null)
        {
            gameTimeline.pause();
        }
    }
    
    public void resumeGame()
    {
        if(flappyJumpTimeline != null && !gravityFlag)
        {
            flappyJumpTimeline.play();
        }
        if(gravityTimeline != null  && gravityFlag)
        {
            gravityTimeline.play();
        }
        if(backgroundTimeline != null)
        {
            backgroundTimeline.play();
        }
        if(pillerTimeline[0] != null)
        {
            for(int i=0; i<4; i++)
            {
                pillerTimeline[i].play();
            }
        }
        if(gameTimeline != null)
        {
            gameTimeline.play();
        }
    }
    
    public void stopGame()
    {
        if(flappyJumpTimeline != null)
        {
            flappyJumpTimeline.stop();
        }
        if(gravityTimeline != null)
        {
            gravityTimeline.stop();
        }
        if(backgroundTimeline != null)
        {
            backgroundTimeline.stop();
        }
        if(pillerTimeline[0] != null)
        {
            for(int i=0; i<4; i++)
            {
                pillerTimeline[i].stop();
            }
        }
        if(gameTimeline != null)
        {
            gameTimeline.stop();
        }
    }
    
    public void flappyFishStartAnimation()
    {
        ImageView flappyImageView = new ImageView(new Image(getClass().getResourceAsStream("/resourcepack/FlappyBird2.png")));
        flappyImageView.setFitWidth(70);
        flappyImageView.setFitHeight(50);
        Label startFlappyLabel = new Label("", flappyImageView);
        
        Text text = new Text("FlappyFX");
        text.setTextOrigin(VPos.TOP);
        text.setEffect(new Reflection());
        text.setFill(Color.BLUE);
        text.setFont(Font.font(100));
        
        Pane startPane = new Pane(text, startFlappyLabel);
        startPane.setPrefSize(screenWidth, screenHeight);
        root.getChildren().add(startPane);
        
        startFlappyLabel.setTranslateY((screenHeight)/2-100);
        startFlappyLabel.setTranslateX(-100);
        text.setTranslateY(startFlappyLabel.getTranslateY());
        text.setTranslateX(screenWidth);
        
        RotateTransition rotateTransition = new RotateTransition(Duration.millis(150), startFlappyLabel);
        rotateTransition.setFromAngle(-45);
        rotateTransition.setToAngle(45);
        rotateTransition.setAxis(Rotate.Y_AXIS);
        rotateTransition.setAutoReverse(true);
        rotateTransition.setCycleCount(50);
        rotateTransition.setInterpolator(Interpolator.EASE_BOTH);
        
        TranslateTransition translateTransition = new TranslateTransition(Duration.seconds(3), startFlappyLabel);
        translateTransition.setFromX(-100);
        translateTransition.setToX((screenWidth-70)/2);
        
        TranslateTransition translateTextTransition = new TranslateTransition(Duration.seconds(3), text);
        translateTextTransition.setFromX(screenWidth);
        translateTextTransition.setToX((screenWidth-text.getLayoutBounds().getWidth())/2);
        
        ParallelTransition parallelTransition = new ParallelTransition(rotateTransition, translateTransition, translateTextTransition);
        parallelTransition.setOnFinished((ae) ->
        {
            root.getChildren().clear();
            createSettingPane();
        });
        parallelTransition.play();
    }
    
    public void addGameOver()
    {
        stopGame();
        gameOverPane = new Pane();
        takeScreenShot();
        root.getChildren().clear();
        stage.getScene().removeEventFilter(KeyEvent.KEY_PRESSED, eventHandler);
        gameOverlabel = new Label("", new ImageView(new Image(getClass().getResourceAsStream("/resourcepack/gameOver.png"), screenWidth, screenHeight,false, true)));
        highestScoreLabel = new Label("Score: "+score.get()+"\n"+"Highest Score: "+getHighestScore());
        highestScoreLabel.setId("highestScoreLabel");
        highestScoreLabel.setAlignment(Pos.CENTER);
        highestScoreLabel.setTextAlignment(TextAlignment.CENTER);
        highestScoreLabel.setPrefSize(500, 200);
        highestScoreLabel.setTranslateX(screenLabel.getTranslateX()+screenLabel.getPrefWidth());
        highestScoreLabel.setTranslateY(screenLabel.getTranslateY()+screenLabel.getPrefHeight()/2);
        gameOverPane.getChildren().addAll(gameOverlabel, screenLabel, highestScoreLabel);
        TranslateTransition translateTransition = new TranslateTransition(Duration.millis(1000), gameOverlabel);
        translateTransition.setFromY(-screenHeight);
        translateTransition.setToY(0);
        translateTransition.setOnFinished((ae) ->
        {
            try
            {
                Thread.sleep(5000);
            }
            catch (Exception e)
            {
                System.err.println("In GameOver: "+e.getMessage());
            }
            root.getChildren().clear();
            root.getChildren().add(settingPane);
            startPauseButton.setText("Start");
            settingPane.setVisible(true);
        });
        root.getChildren().addAll(gameOverPane);
        AudioClip media = new AudioClip(getClass().getResource("/resourcepack/gameover.wav").toExternalForm());
        media.play(volumeControlSlider.getValue());
        translateTransition.play();
    }

    public void initJumpMedia()
    {
        AudioClip jumpPlayer = new AudioClip(getClass().getResource("/resourcepack/flap.wav").toExternalForm());
        jumpPlayer.play(volumeControlSlider.getValue());
    }
    
    public void initDeadMedia()
    {
        AudioClip deadPlayer = new AudioClip(getClass().getResource("/resourcepack/smack.wav").toExternalForm());
        deadPlayer.play(volumeControlSlider.getValue());
    }
    
    public void initSetup()
    {
        spacingHeight = pillerSpacingSlider.getValue();
        flappyBirdJump = flappyJumpHeightSlider.getValue();
        
        ImageView flappyImageView = new ImageView(new Image(getClass().getResourceAsStream("/resourcepack/FlappyBird2.png")));
        flappyImageView.setFitWidth(80);
        flappyImageView.setFitHeight(50);
        
        flappyLabel = new Label("", flappyImageView);
        flappyBoundCircle = new Ellipse(35, 23);
        flappyBoundCircle.setFill(Color.TRANSPARENT);
        //flappyBoundCircle.rotateProperty().bind(flappyLabel.rotateProperty());
        flappyBoundCircle.translateXProperty().bind(flappyLabel.translateXProperty().add(flappyImageView.getFitWidth()/2));
        flappyBoundCircle.translateYProperty().bind(flappyLabel.translateYProperty().add(flappyImageView.getFitHeight()/2));
        System.out.println(flappyLabel.getBoundsInParent().getWidth()/2);
        flappyImageView = new ImageView(new Image(getClass().getResourceAsStream("/resourcepack/Back.png"), screenWidth, screenHeight,false, true));
        backgroundLabel = new Label("", flappyImageView);
        
        score = new SimpleIntegerProperty(0);
        score.addListener((pc, ov, nv) ->
        {
            scoreLabel.setText("Score: "+score.get());
        });
        
        scoreLabel = new Label("Score: "+score.getValue());
        scoreLabel.setId("scoreLabel");
        scoreLabel.setPrefSize(400, 70);
        scoreLabel.setAlignment(Pos.CENTER);
        scoreLabel.setTranslateX(20);
        scoreLabel.setTranslateY(20);
        
        lifeBar = new ProgressBar(0);
        lifeBar.setPrefSize(40*(int)lifeSlider.getValue(), 50);
        lifeBar.setTranslateY(scoreLabel.getTranslateY()+scoreLabel.getPrefHeight()+10);
        lifeBar.setTranslateX(scoreLabel.getTranslateX());
        lifeBar.progressProperty().addListener((pc, ov, nv) ->
        {
            if(nv.doubleValue() > 0.9)
            {
                addGameOver();
            }
        });
        
        pillerLabel = new Label[8];
        pillerTimeline = new Timeline[4];
        pillerStrike = new boolean[]{true,true,true,true};
        for(int i=0; i<pillerLabel.length; i++)
        {
            pillerLabel[i] = new Label("", new ImageView(new Image(getClass().getResourceAsStream("/resourcepack/walls4.png"),60 , 600, false, true)));
            pillerLabel[i].setTranslateX(3000);
        }
        
        flappyLabel.setTranslateX(screenWidth/3);
        flappyLabel.setTranslateY(200);
        backgroundLabel.setTranslateX(0);
        backgroundLabel.setTranslateY(0);
        
        eventHandler = (KeyEvent ke) ->
        {
            if((ke.getCode() == KeyCode.UP || ke.getCode() == KeyCode.SPACE || ke.getCode() == KeyCode.CONTROL) && !settingPane.isVisible())
            {
                jumpAnimation();
                initJumpMedia();
            }
            if(ke.getCode() == KeyCode.ESCAPE)
            {
                if(settingPane.isVisible())
                {
                    settingPane.setVisible(false);
                    resumeGame();
                    settingPane.toBack();
                    startPauseButton.setText("Stop");
                }
                else
                {
                    pauseGame();
                    settingPane.setVisible(true);
                    settingPane.toFront();
                    startPauseButton.setText("Resume");
                }
            }
        };
        
        int delay  = 0;
        aa=0;
        for(int i=0; i<pillerLabel.length; i+=2)
        {
            pillerLabel[i].translateXProperty().bindBidirectional(pillerLabel[i+1].translateXProperty());
            int y = random.nextInt(400);
            while(y < pillerSpacingSlider.getValue())
            {
                y = random.nextInt(400);
            }
            pillerLabel[i].setTranslateY(-y);
            pillerLabel[i+1].setTranslateY(pillerLabel[i].getTranslateY()+spacingHeight+600);
            pillerAnimation(i, delay);
            delay += 1200;
        }
        
        KeyFrame gameFrame = new KeyFrame(Duration.millis(10), this::gameLoop);
        gameTimeline = new Timeline(gameFrame);
        gameTimeline.setCycleCount(Timeline.INDEFINITE);
        gameTimeline.play();
        gravityAnimation();
    }
    
    public void gameLoop(ActionEvent ae)
    {
        scoreLabel.setStyle("-fx-background-color: green");
        Bounds bounds = flappyBoundCircle.getBoundsInParent();
        if(bounds.getMinY() <= 0)
        {
            flappyJumpTimeline.stop();
            gravityAnimation();
        }
        if(bounds.intersects(pillerLabel[0].getBoundsInParent()) || bounds.intersects(pillerLabel[1].getBoundsInParent()))
        {
            hurtAnimation();
        }
        else if(bounds.intersects(pillerLabel[2].getBoundsInParent()) || bounds.intersects(pillerLabel[3].getBoundsInParent()))
        {
            hurtAnimation();
        }
        else if(bounds.intersects(pillerLabel[4].getBoundsInParent()) || bounds.intersects(pillerLabel[5].getBoundsInParent()))
        {
            hurtAnimation();
        }
        else if(bounds.intersects(pillerLabel[6].getBoundsInParent()) || bounds.intersects(pillerLabel[7].getBoundsInParent()))
        {
            hurtAnimation();
        }
        else
        {
            if(bounds.getMaxX() >= pillerLabel[0].getBoundsInParent().getMinX() && pillerStrike[0])
            {
                pillerStrike[0] = false;
                incScore();
            }
            if(bounds.getMaxX() >= pillerLabel[2].getBoundsInParent().getMinX() && pillerStrike[1])
            {
                pillerStrike[1] = false;
                incScore();
            }
            if(bounds.getMaxX() >= pillerLabel[4].getBoundsInParent().getMinX() && pillerStrike[2])
            {
                pillerStrike[2] = false;
                incScore();
            }
            if(bounds.getMaxX() >= pillerLabel[6].getBoundsInParent().getMinX() && pillerStrike[3])
            {
                pillerStrike[3] = false;
                incScore();
            }
        }
    }
    
    public void incScore()
    {
        initCoinMedia();
    }
    
    public void initCoinMedia()
    {
        AudioClip coinPlayer = new AudioClip(getClass().getResource("/resourcepack/Coin_Sound.wav").toExternalForm());
        coinPlayer.play(volumeControlSlider.getValue());
        score.set(score.get()+1);
    }
    
    public void hurtAnimation()
    {
        scoreLabel.setStyle("-fx-background-color: red");
        if(fadeTransition != null && fadeTransition.getStatus() == FadeTransition.Status.RUNNING)
        {
            return;
        }
        lifeBar.setProgress(lifeBar.getProgress()+(1/lifeSlider.getValue()));
        initDeadMedia();
        fadeTransition = new FadeTransition(Duration.millis(50), flappyLabel);
        fadeTransition.setFromValue(0.0);
        fadeTransition.setToValue(1.0);
        fadeTransition.setCycleCount(15);
        fadeTransition.play();
    }
    
    public void takeScreenShot()
    {
        SnapshotParameters snapshotParameters = new SnapshotParameters();
        Rectangle2D rectangle2D = new Rectangle2D(flappyLabel.getTranslateX()-200, flappyLabel.getTranslateY()-200, 400, 400);
        snapshotParameters.setViewport(rectangle2D);
        WritableImage writableImage = root.snapshot(snapshotParameters, null);
        screenLabel = new Label("", new ImageView(writableImage));
        screenLabel.setAlignment(Pos.CENTER);
        screenLabel.setRotate(20);
        screenLabel.setPrefSize(420,420);
        screenLabel.setStyle("-fx-background-color: white;");
        screenLabel.setTranslateX((screenWidth-screenLabel.getPrefWidth())/4);
        screenLabel.setTranslateY((screenHeight-screenLabel.getPrefHeight())/2);
    }
    
    public int getHighestScore()
    {
        highestScore = new SimpleIntegerProperty(0);
        try
        {
            File file = new File("setting.ini");
            if(file.exists())
            {
                RandomAccessFile raf = new RandomAccessFile(file, "r");
                highestScore.set(raf.readInt());
                raf.close();
            }
            else
            {
                highestScore.set(0);
            }
            if(score.get() <= highestScore.get())
            {
                return highestScore.get();
            }
            if(file.exists())
            {
                file.delete();
            }
            RandomAccessFile raf = new RandomAccessFile(file, "rws");
            raf.writeInt(score.get());
            raf.close();
        }
        catch (Exception e)
        {
            System.out.println("IO Error: "+ e.getMessage());
        }
        return score.get();
    }
    
    public static void main(String[] args)
    {
        launch(args);
    }
}
