# Progreso-Space-ivanders
Progreso 3 interactividad space ivanders
/* @pjs preload="bala.png"; */
/* @pjs preload="bitxo1.png"; */
/* @pjs preload="bitxo2.png"; */
/* @pjs preload="nau.png"; */
//////////////////////
//  VARIABLES PRINCIPALES
//
//------PANTALLA------------:
//
//Ancho de la pantalla
int theWidth = 600;
//Alto de la pantalla
int theHeight = 400;
//
//------INVASORES------------:
//
//velocidad de los invasores
float invadersSpeed = 1;
//incremento de la velocidad cada vez que cambian de direcciÃ³n
float invadersSpeedIncrement = 0.05;
//pixels que bajan cada vez que cambian de direcciÃ³n
int invadersYStep = 4;
//
//------NAVE------------:
//
//distancia de la nave al borde inferior de la pantalla
int spaceShipDistanceToBottom = 25;
//velocidad a la que se mueve la nave
int spaceShipSpeed = 5;
//
//------BALAS------------:
//
//Velocidad a la que van las balas
int bulletSpeed = 4;
//Tiempo (en milisegundos) que ha de pasar desde que se disparÃ³ una bala
//hasta que se puede disparar otra
int delayBetweenBullets = 500;
//
//////////////////////////////////////////////////
//cbjetos p
int numOfInvaders = 50;
invader[] invaders = new invader[numOfInvaders];
spaceShip nave = new spaceShip(theWidth/2, theHeight-spaceShipDistanceToBottom,spaceShipSpeed, delayBetweenBullets);
//imÃ¡genes
PImage spaceShip, bulletImage,invadersFrameOne,invadersFrameTwo;
ArrayList bulletsList = new ArrayList();


void setup(){
  //size(theWidth,theHeight);
  size(600,400);
  imageMode(CENTER);
  rectMode(CENTER);
  //cargamos imÃ¡genes
  spaceShip = loadImage("nau.gif");
  invadersFrameOne = loadImage("bitxo1.gif");
  invadersFrameTwo = loadImage("bitxo2.gif");
  bulletImage = loadImage("bala.gif");
  spaceShipSpeed = 5;
  bulletSpeed = 4;
  //INICIALIZACION (esto funciona para 50 invasores a 10x5)
  int invaderCount = 0;
  for(int i=50;i<200;i+=30){
    for(int j=75;j<550;j+=50){
      invaders[invaderCount]=new invader(j,i,invaderCount, invadersSpeed, invadersSpeedIncrement, invadersYStep);
      invaderCount++;
    }
  }

}

void draw(){
  background(0);
  for(int i=0;i<numOfInvaders;i++){
    invaders[i].update();
  }
  nave.update();
  if(bulletsList.size()>0){
    for(int i=0; i<bulletsList.size();i++){
      bullet _b = (bullet) bulletsList.get(i);
      _b.update(); 
    }
  }
}
///////////CONTROL CON TECLADO
//cuando le damos a una tecla
void keyPressed(){
  //miramos si es de las raras:
  if (key == CODED) {
    //y si lo es, si es la flecha izquierda o derecha
    if (keyCode == LEFT) {
      nave.decrementX();
    } 
    else if (keyCode == RIGHT) { 
      nave.incrementX();
    } 
  } 
  else {
    //si le damos a la tecla espacio
    if(key==' '){
      nave.shoot();
    } 
  }
}




class bullet{
  float x,y,speed;
  bullet(float _x, float _s){
    x=_x;
    y=height-33;
    speed = _s;
  }
  void update(){
    move();
    checkInvaders(); 
    checkUpperBorder();
    drawMe();
  }
  void move(){
    y -= speed;
  }

  void checkInvaders(){
    for(int i=0;i<numOfInvaders;i++){
      if(invaders[i].isAlive()){
        float ix = invaders[i].getX();
        float iy = invaders[i].getY();
        //teniendo en cuenta que los invaders son imÃ¡genes de 24x16
        if(abs(ix-x)<12 && abs(iy-y)<8){
          invaders[i].kill();
          removeMe();
        }
      }
    }
  }

  void checkUpperBorder(){
    if(y<0){
      removeMe();
    }  
  }

  void drawMe(){
    image(bulletImage,x,y);
  }

  void removeMe(){
    bulletsList.remove(this); 
  }
}



class invader{
  float x,y,originX,originY;
  int dir=1;
  float speed, speedIncrement;
  int yStep;
  int distanceAllowed = 60;
  boolean alive = true ;
  int id = -1;
  int myTime = 0;
  int frameInterval = 200;
  boolean frameOne = true;

  invader(int _x, int _y, int _id, float _s, float _si, int _ys){
    x=originX=_x;
    y=originY=_y;
    id = _id;
    speed = _s;
    speedIncrement = _si;
    yStep = _ys;
  } 

  void update(){
    if(alive){
      move();
      drawMe(); 
    }
  }

  void move(){
    x += speed*dir;
    //if(id==20){
    //  println(x+"__"+originX);
    //  println((abs(originX-x)>distanceAllowed));
    //println("x="+x+"  s+d:"+speed*dir);  
    //}

    //Scambio de direccion
    if(abs(x-originX)>distanceAllowed){
      dir = -dir;
      y += yStep; 
      speed += speedIncrement;
    }
  }

  void drawMe(){
    if(millis() - myTime > frameInterval){
      myTime = millis();
      frameOne = !frameOne;
    }
    if(frameOne){
      image(invadersFrameOne,x,y);
    } 
    else {
      image(invadersFrameTwo,x,y);
    }
  }

  //GETs y SETs
  void setX(int _x){
    x=_x;
  }
  void setY(int _y){
    y=_y;
  }
  float getX(){
    return x;
  }
  float getY(){
    return y;
  }
  void kill(){
    alive = false; 
  }
  boolean isAlive(){
    if(alive)return true;
    else return false;
  }

}





class spaceShip{
  int x, y;
  int xStep, delayBetweenBullets;
  int lastBulletTime = -10000;
  spaceShip(int _x, int _y, int _s, int _d){
    x = _x;
    y = _y;
    xStep = _s;
    delayBetweenBullets = _d;
  } 

  void update(){
    drawMe(); 
    checkInvaders();
  }

  void drawMe(){
    //println("_____________"+x+"oo"+y);
    //image(spaceShip,x,y);
    
    fill(0,255,0);
    stroke(0,255,0);
    rect(x,y+6,30,10);
    rect(x,y,4,12);
  }

  void incrementX(){
    x += xStep;
  }
  void decrementX(){
    x -= xStep;
  }
  void setX(int _newX){
    x = _newX; 
  }

  void shoot(){
    if(millis()-lastBulletTime > delayBetweenBullets){ 
      bulletsList.add(new bullet(x,bulletSpeed));
      lastBulletTime = millis();
    }
  } 

  void checkInvaders(){
    for(int i=0;i<numOfInvaders;i++){
      if(invaders[i].isAlive()){
        float iy = invaders[i].getY();
        if(iy >= height-spaceShipDistanceToBottom-15){
          // fin JUEGO
          fill(255,0,0);
          stroke(255,0,0);
          rect(0,0,width,height);
        }
      }
    }
  }
}
