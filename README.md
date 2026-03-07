/* --------------------------------
   FRACTAL SULI KOLAM
   GRAMMAR BASED
   NO ARCS
   CLOSED LOOP
-------------------------------- */

String axiom;
String instructions;

int iterations;

float forwardLen = 20;

float xPos, yPos;
float angleDeg = 0;

int drawIndex = 0;

/* DOT GRID */

int grid = 15;
float spacing = 40;

float minX, maxX, minY, maxY;

/* PATH */

ArrayList<PVector> path;

/* COLORS */

color[] colors = {
#E53935,
#1E88E5,
#43A047,
#8E24AA
};

int colorIndex = 0;
color currentColor;

/* ---------- SETUP ---------- */

void setup(){

  fullScreen();
  background(255);

  strokeWeight(3);
  frameRate(120);
  noFill();

  computeBounds();
  newKolam();
}

/* ---------- GRID BOUNDS ---------- */

void computeBounds(){

  float startX = width/2 - (grid/2)*spacing;
  float startY = height/2 - (grid/2)*spacing;

  minX = startX;
  maxX = startX + grid*spacing;

  minY = startY;
  maxY = startY + grid*spacing;
}

/* ---------- DRAW ---------- */

void draw(){

  drawDots();

  if(drawIndex >= instructions.length()){
    drawSymmetry();
    return;
  }

  char ch = instructions.charAt(drawIndex);

  if(ch=='F'){
    moveForward(forwardLen);
  }

  else if(ch=='+'){
    angleDeg += 90;
    changeColor();
  }

  else if(ch=='-'){
    angleDeg -= 90;
    changeColor();
  }

  drawIndex++;
}

/* ---------- DOTS ---------- */

void drawDots(){

  stroke(0);
  strokeWeight(6);

  float startX = width/2 - (grid/2)*spacing;
  float startY = height/2 - (grid/2)*spacing;

  for(int i=0;i<grid;i++){
    for(int j=0;j<grid;j++){

      point(startX + i*spacing,
            startY + j*spacing);
    }
  }

  strokeWeight(3);
}

/* ---------- NEW PATTERN ---------- */

void newKolam(){

  background(255);

  path = new ArrayList<PVector>();

  drawIndex = 0;
  colorIndex = 0;

  xPos = width/2;
  yPos = height/2;

  angleDeg = 0;

  path.add(new PVector(xPos,yPos));

  currentColor = colors[0];

  String[] axioms = {

  "F+F+F+F",
  "F+F-F+F",
  "F-F+F-F",
  "F+F+F-F"

  };

  axiom = axioms[int(random(axioms.length))];

  iterations = int(random(3,5));

  instructions = generateLSystem(axiom,iterations);
}

/* ---------- L SYSTEM ---------- */

String generateLSystem(String ax,int iter){

  String current = ax;

  for(int i=0;i<iter;i++){

    String next="";

    for(int j=0;j<current.length();j++){

      char ch = current.charAt(j);

      if(ch=='F'){

        String[] rules={

        "F+F-F-F+F",
        "F-F+F+F-F",
        "F+F+F-F",
        "F-F-F+F"

        };

        next += rules[int(random(rules.length))];
      }

      else{
        next += ch;
      }
    }

    current = next;
  }

  return current;
}

/* ---------- MOVE ---------- */

void moveForward(float step){

  float nx = xPos + cos(radians(angleDeg))*step;
  float ny = yPos + sin(radians(angleDeg))*step;

  if(nx<minX || nx>maxX || ny<minY || ny>maxY){
    return;
  }

  stroke(currentColor);

  line(xPos,yPos,nx,ny);

  xPos = nx;
  yPos = ny;

  path.add(new PVector(xPos,yPos));
}

/* ---------- SYMMETRY ---------- */

void drawSymmetry(){

  float cx = width/2;
  float cy = height/2;

  for(int r=1;r<4;r++){

    float ang = radians(r*90);

    for(int i=path.size()-1;i>0;i--){

      PVector p1 = path.get(i);
      PVector p2 = path.get(i-1);

      float x1 = p1.x - cx;
      float y1 = p1.y - cy;

      float x2 = p2.x - cx;
      float y2 = p2.y - cy;

      float rx1 = x1*cos(ang) - y1*sin(ang);
      float ry1 = x1*sin(ang) + y1*cos(ang);

      float rx2 = x2*cos(ang) - y2*sin(ang);
      float ry2 = x2*sin(ang) + y2*cos(ang);

      line(cx+rx1,cy+ry1,
           cx+rx2,cy+ry2);
    }
  }
}

/* ---------- COLOR CHANGE ---------- */

void changeColor(){

  currentColor = colors[colorIndex];
  colorIndex = (colorIndex+1) % colors.length;
}

/* ---------- TOUCH ---------- */

void mousePressed(){

  newKolam();
}
