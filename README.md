DDA.cpp
#include <GL/glut.h>
#include <iostream>
#include <cmath>

using namespace std;

int startX, startY, endX, endY;  // Points for the line
bool drawingLine = false;  // Flag to check if the user is drawing a line
int lineType = 0;  // 0: Simple Line, 1: Dotted, 2: Dashed, 3: Solid

// Function to draw the center axis lines
void drawAxis() {
    glColor3f(0, 0, 0);  // Black color for the axis
    glLineWidth(1.0f);  // Default thickness for the axis lines
    glBegin(GL_LINES);
        glVertex2i(0, -300);
        glVertex2i(0, 300);  // Y-axis
        glVertex2i(-300, 0);
        glVertex2i(300, 0);  // X-axis
    glEnd();
}

// Function to set line color (darker color)
    void setLineColor() {
    glColor3f(0.2f, 0.2f, 0.2f);  // Keep the dark gray color
    glPointSize(2.0f);  // Thinner point size
}


// DDA Line Drawing Algorithm
void drawDDA(int x1, int y1, int x2, int y2) {
    float dx = x2 - x1;
    float dy = y2 - y1;
    int steps;
    float xIncrement, yIncrement;
    
    // Calculate the number of steps required for generating the points
    if (abs(dx) > abs(dy)) {
        steps = abs(dx);
    } else {
        steps = abs(dy);
    }
    
    // Calculate the increment for each step
    xIncrement = dx / steps;
    yIncrement = dy / steps;

    // Initial point
    float x = x1;
    float y = y1;

    // Set the line color (darker color)
    setLineColor();

    // Set the line width for the drawing lines (thicker lines)
    glLineWidth(2.0f);  // Make the drawing lines thicker

    // Draw the points
    for (int i = 0; i <= steps; i++) {
        if (lineType == 0 || (lineType == 1 && i % 4 == 0) || (lineType == 2 && i % 8 == 0) || lineType == 3) {
            glBegin(GL_POINTS);
            glVertex2i(round(x), round(y));  // Plot the point
            glEnd();
        }
        x += xIncrement;
        y += yIncrement;
    }
}

// Bresenham's Line Drawing Algorithm
void drawBresenham(int x1, int y1, int x2, int y2) {
    int dx = abs(x2 - x1);
    int dy = abs(y2 - y1);
    int p = 2 * dy - dx;
    int twoDy = 2 * dy;
    int twoDyMinusDx = 2 * (dy - dx);
    int x = x1, y = y1;

    // Set the line color (darker color)
    setLineColor();

    // Set the line width for the drawing lines (thicker lines)
    glLineWidth(2.0f);  // Make the drawing lines thicker

    // Draw the first point
    glBegin(GL_POINTS);
    glVertex2i(x, y);
    glEnd();

    if (dx > dy) {
        if (x1 > x2) {
            swap(x1, x2);
            swap(y1, y2);
        }
        for (x = x1 + 1; x <= x2; x++) {
            if (p < 0) {
                p += twoDy;
            } else {
                y = (y1 < y2) ? y + 1 : y - 1;
                p += twoDyMinusDx;
            }

            if (lineType == 0 || (lineType == 1 && x % 4 == 0) || (lineType == 2 && x % 8 == 0) || lineType == 3) {
                glBegin(GL_POINTS);
                glVertex2i(x, y);  // Plot the point
                glEnd();
            }
        }
    } else {
        if (y1 > y2) {
            swap(x1, x2);
            swap(y1, y2);
        }
        for (y = y1 + 1; y <= y2; y++) {
            if (p < 0) {
                p += 2 * dx;
            } else {
                x = (x1 < x2) ? x + 1 : x - 1;
                p += 2 * (dx - dy);
            }

            if (lineType == 0 || (lineType == 1 && y % 4 == 0) || (lineType == 2 && y % 8 == 0) || lineType == 3) {
                glBegin(GL_POINTS);
                glVertex2i(x, y);  // Plot the point
                glEnd();
            }
        }
    }
}
// Display function (empty, we are using other methods to rende
void display() {}

// Function to handle mouse clicks
void mouseClick(int button, int state, int x, int y) {
    if (button == GLUT_LEFT_BUTTON && state == GLUT_DOWN) {
        // Convert the mouse coordinates to OpenGL coordinates (centered at (0, 0))
        x = x - 300;  // Shifting the origin to the center
        y = 300 - y;  // Flip the y-axis to match OpenGL coordinates
        
        if (!drawingLine) {
            startX = x;
            startY = y;
            drawingLine = true;  // Now we are defining the second point
        } else {
            endX = x;
            endY = y;
            glClear(GL_COLOR_BUFFER_BIT);  // Clear the screen
            drawAxis();                    // Draw the axes
     
        if (lineType == 0) 
            drawDDA(startX, startY, endX, endY);
         else 
            drawBresenham(startX, startY, endX, endY);
        
       drawingLine = false;  // Line drawn, reset the flag
      }
    }
    glFlush();
}

// Function to handle keyboard input (for selecting line types)
void keyboard(unsigned char key, int x, int y) {
    cout << "Key pressed: " << key << endl;  
    if (key == '1') {
        lineType = 0;  // Simple Line
        cout << "Simple Line selected" << endl;  // Debugging output
    } else if (key == '2') {
        lineType = 1;  // Dotted Line
        cout << "Dotted Line selected" << endl;  
    } else if (key == '3') {
        lineType = 2;  // Dashed Line
        cout << "Dashed Line selected" << endl;  
    } else if (key == '4') {
        lineType = 3;  // Solid Line
        cout << "Solid Line selected" << endl; 
    }
}

// Function to initialize OpenGL
void initOpenGL() {
    glClearColor(1, 1, 1, 1);  
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(-300, 300, -300, 300);  // Set the clipping area
    
    // Set the line width to a thinner value for the axes (default is 1.0)
    glLineWidth(1.0f);  // Keep the axis lines at default thickness (1.0)
}


int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(600, 600);  // Set window size
    glutCreateWindow("Line Drawing Algorithms");

    initOpenGL();

    glutDisplayFunc(display);  // Register display callback
    glutMouseFunc(mouseClick);  // Register mouse click callback
    glutKeyboardFunc(keyboard);  // Register keyboard callback

    glutMainLoop();  // Start the GLUT main loop
    return 0;
}














































Circle.cpp
#include <GL/glut.h>
#include <cmath>
#include <iostream>

// Global variables for center, radius, and selected quadrant
int centerX = 0, centerY = 0;
int radius = 0;
int selectedQuadrant = 0; // 1: Q1, 2: Q2, 3: Q3, 4: Q4
bool isCenterSet = false;  // Flag to check if center is set
bool isRadiusSet = false;  // Flag to check if radius is set

// Function to plot points at the given (x, y)
void plotPoint(int x, int y) {
    glBegin(GL_POINTS);
    glVertex2i(x, y);
    glEnd();
}

// Bresenham's Circle Drawing Algorithm
void bresenhamCircle(int radius) {
    int x = 0;
    int y = radius;
    int p = 3 - 2 * radius;

    // Draw the circle in the selected quadrant
    while (x <= y) {
        if (selectedQuadrant == 1) { // Quadrant 1
            plotPoint(centerX + x, centerY + y);
            plotPoint(centerX - x, centerY + y);
            plotPoint(centerX + x, centerY - y);
            plotPoint(centerX - x, centerY - y);
            plotPoint(centerX + y, centerY + x);
            plotPoint(centerX - y, centerY + x);
            plotPoint(centerX + y, centerY - x);
            plotPoint(centerX - y, centerY - x);
        } else if (selectedQuadrant == 2) { // Quadrant 2
            plotPoint(centerX - x, centerY + y);
            plotPoint(centerX + x, centerY + y);
            plotPoint(centerX - x, centerY - y);
            plotPoint(centerX + x, centerY - y);
            plotPoint(centerX - y, centerY + x);
            plotPoint(centerX + y, centerY + x);
            plotPoint(centerX - y, centerY - x);
            plotPoint(centerX + y, centerY - x);
        } else if (selectedQuadrant == 3) { // Quadrant 3
            plotPoint(centerX - x, centerY - y);
            plotPoint(centerX + x, centerY - y);
            plotPoint(centerX - x, centerY + y);
            plotPoint(centerX + x, centerY + y);
            plotPoint(centerX - y, centerY - x);
            plotPoint(centerX + y, centerY - x);
            plotPoint(centerX - y, centerY + x);
            plotPoint(centerX + y, centerY + x);
        } else if (selectedQuadrant == 4) { // Quadrant 4
            plotPoint(centerX + x, centerY - y);
            plotPoint(centerX - x, centerY - y);
            plotPoint(centerX + x, centerY + y);
            plotPoint(centerX - x, centerY + y);
            plotPoint(centerX + y, centerY - x);
            plotPoint(centerX - y, centerY - x);
            plotPoint(centerX + y, centerY + x);
            plotPoint(centerX - y, centerY + x);
        }
        if (p < 0) {
            p = p + 4 * x + 6;
        } else {
            p = p + 4 * (x - y) + 10;
            y--;
        }
        x++;
    }
}

// Function to draw the division lines (quadrants)
void drawQuadrants() {
    glColor3f(1.0f, 1.0f, 1.0f);  // Set color to white

    // Draw the vertical and horizontal lines to divide the window into 4 quadrants
    glBegin(GL_LINES);
        glVertex2i(0, 300); glVertex2i(0, -300);  // Vertical line
        glVertex2i(-300, 0); glVertex2i(300, 0);  // Horizontal line
    glEnd();
}

// Function to handle mouse clicks and capture center, radius, and quadrant selection
void mouseFunc(int button, int state, int x, int y) {
    if (button == GLUT_LEFT_BUTTON && state == GLUT_DOWN) {
        int window_width = 600;
        int window_height = 600;
        int screenCenterX = window_width / 2;
        int screenCenterY = window_height / 2;

        // Convert mouse click position to OpenGL coordinates
        int openglY = screenCenterY - y;  // Adjust for OpenGL coordinate system

        if (!isCenterSet) {
            // First click to set the center of the circle
            centerX = x - screenCenterX;  // Translate click to OpenGL coordinates
            centerY = openglY;
            isCenterSet = true; // Center has been set
            std::cout << "Center Set at: (" << centerX << ", " << centerY << ")\n";
        } else if (!isRadiusSet) {
            // Second click to set the radius of the circle
            int dx = x - screenCenterX - centerX;  // Distance from center in x
            int dy = openglY - centerY;            // Distance from center in y
            radius = static_cast<int>(sqrt(dx * dx + dy * dy));  // Calculate the distance (radius)
            isRadiusSet = true; // Radius has been set
            std::cout << "Radius Set: " << radius << "\n";
        } else {
            // Third click to select the quadrant (divide the window into 4 quadrants)
            if (x < screenCenterX && y > screenCenterY) {
                selectedQuadrant = 2;  // Quadrant 2 (top-left)
            } else if (x > screenCenterX && y > screenCenterY) {
                selectedQuadrant = 1;  // Quadrant 1 (top-right)
            } else if (x < screenCenterX && y < screenCenterY) {
                selectedQuadrant = 3;  // Quadrant 3 (bottom-left)
            } else if (x > screenCenterX && y < screenCenterY) {
                selectedQuadrant = 4;  // Quadrant 4 (bottom-right)
            }

            // Redraw the window after setting the quadrant
            glutPostRedisplay();
        }
    }
}

// Function to handle keyboard input
void keyboardFunc(unsigned char key, int x, int y) {
    if (key == 'r' || key == 'R') {
        // Reset center, radius, and selected quadrant on 'r' key press
        isCenterSet = false;
        isRadiusSet = false;
        selectedQuadrant = 0;
        radius = 0;
        centerX = 0;
        centerY = 0;
        std::cout << "Resetting Circle\n";  // Inform user of reset
        glutPostRedisplay();  // Redraw the window
    }
}

// Display function to set up the OpenGL window
void display() {
    glClear(GL_COLOR_BUFFER_BIT);  // Clear the screen

    // Draw the quadrants first
    drawQuadrants();

    // Draw the circle if center and radius are set
    glColor3f(1.0f, 1.0f, 1.0f);  // Set color to white
    if (isCenterSet && isRadiusSet && selectedQuadrant != 0) {
        bresenhamCircle(radius);  // Draw the circle in the selected quadrant
    }

    glFlush();  // Render the scene
}

// Initialize OpenGL settings
void initOpenGL() {
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);  // Set background color to black
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    glOrtho(-300, 300, -300, 300, -1.0, 1.0);  // Set orthographic view with origin at center
    glPointSize(2.0);  // Set point size
}

// Main function
int main(int argc, char **argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(600, 600);
    glutCreateWindow("Bresenham Circle Drawing with Mouse & Quadrant Selection");

    initOpenGL();  // Initialize OpenGL settings
    glutDisplayFunc(display);  // Set display callback function
    glutMouseFunc(mouseFunc);  // Set mouse click callback function
    glutKeyboardFunc(keyboardFunc);  // Set keyboard input callback function
    glutMainLoop();  // Enter the main loop

    return 0;
}

























Polygon_Filling.cpp
#include <GL/glut.h>
#include <iostream>
using namespace std;

// Fill color values
float R = 0, G = 0, B = 0;
// 1 = Boundary Fill, 2 = Flood Fill
int Algo;

// Initialize OpenGL
void init() {
    glClearColor(1.0, 1.0, 1.0, 0.0);
    glMatrixMode(GL_PROJECTION);
    gluOrtho2D(0, 640, 0, 480);
}

// Flood Fill algorithm
void floodFill(int x, int y, float* fillColor, float* oldColor) {
    float color[3];
    glReadPixels(x, y, 1, 1, GL_RGB, GL_FLOAT, color);

    if (color[0] == oldColor[0] && color[1] == oldColor[1] && color[2] == oldColor[2]) {
        glColor3f(fillColor[0], fillColor[1], fillColor[2]);
        glBegin(GL_POINTS);
        glVertex2i(x, y);
        glEnd();
        glFlush();

        floodFill(x + 1, y, fillColor, oldColor);
        floodFill(x - 1, y, fillColor, oldColor);
        floodFill(x, y + 1, fillColor, oldColor);
        floodFill(x, y - 1, fillColor, oldColor);
    }
}

// Boundary Fill algorithm
void boundaryFill(int x, int y, float* fillColor, float* boundaryColor) {
    float color[3];
    glReadPixels(x, y, 1, 1, GL_RGB, GL_FLOAT, color);

    bool notBoundary = (color[0] != boundaryColor[0] || color[1] != boundaryColor[1] || color[2] != boundaryColor[2]);
    bool notFilled = (color[0] != fillColor[0] || color[1] != fillColor[1] || color[2] != fillColor[2]);

    if (notBoundary && notFilled) {
        glColor3f(fillColor[0], fillColor[1], fillColor[2]);
        glBegin(GL_POINTS);
        glVertex2i(x, y);
        glEnd();
        glFlush();

        boundaryFill(x + 1, y, fillColor, boundaryColor);
        boundaryFill(x - 1, y, fillColor, boundaryColor);
        boundaryFill(x, y + 1, fillColor, boundaryColor);
        boundaryFill(x, y - 1, fillColor, boundaryColor);
    }
}

// Mouse click callback
void mouse(int button, int state, int x, int y) {
    y = 480 - y;  // Convert to OpenGL coordinate system

    if (button == GLUT_LEFT_BUTTON && state == GLUT_DOWN) {
        float boundaryColor[] = {1, 0, 0};    // Red border
        float oldColor[] = {1, 1, 1};         // Background white
        float fillColor[] = {R, G, B};        // User-selected color

        if (Algo == 1) boundaryFill(x, y, fillColor, boundaryColor);
        if (Algo == 2) floodFill(x, y, fillColor, oldColor);
    }
}

// Shape for Boundary Fill
void drawBoundaryShape() {
    glClear(GL_COLOR_BUFFER_BIT);
    glColor3f(1, 0, 0);  // Red boundary
    glBegin(GL_LINE_LOOP);
        glVertex2i(150, 100);
        glVertex2i(300, 300);
        glVertex2i(450, 100);
    glEnd();
    glFlush();
}

// Shape for Flood Fill
void drawFloodShape() {
    glClear(GL_COLOR_BUFFER_BIT);
    glBegin(GL_LINES);
        glColor3f(1, 0, 0); glVertex2i(150, 100); glVertex2i(300, 300);
    glEnd();
    glBegin(GL_LINE_LOOP);
        glColor3f(0, 0, 1); glVertex2i(300, 300); glVertex2i(450, 100);
    glEnd();
    glBegin(GL_LINE_LOOP);
        glColor3f(0, 0, 0); glVertex2i(450, 100); glVertex2i(150, 100);
    glEnd();
    glFlush();
}

// Color menu
void colorMenu(int option) {
    switch (option) {
        case 1: R = 0; G = 1; B = 0; break;  // Green
        case 2: R = 1; G = 1; B = 0; break;  // Yellow
        case 3: R = 0; G = 0; B = 1; break;  // Blue
    }
    glutPostRedisplay();
}

int main(int argc, char** argv) {
    cout << "\nSelect the Algorithm:\n1. Boundary Fill\n2. Flood Fill\nChoice: ";
    cin >> Algo;

    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(640, 480);
    glutInitWindowPosition(200, 200);
    glutCreateWindow("Boundary Fill and Flood Fill");

    init();

    // Menu for selecting fill color
    glutCreateMenu(colorMenu);
    glutAddMenuEntry("Color 1: Green", 1);
    glutAddMenuEntry("Color 2: Yellow", 2);
    glutAddMenuEntry("Color 3: Blue", 3);
    glutAttachMenu(GLUT_RIGHT_BUTTON);

    if (Algo == 1) glutDisplayFunc(drawBoundaryShape);
    else if (Algo == 2) glutDisplayFunc(drawFloodShape);

    glutMouseFunc(mouse);
    glutMainLoop();

    return 0;
}







































Polygon_Clipping.cpp
#include <GL/glut.h>
#include <iostream>
#include <vector>

using namespace std;

struct Point {
    int x, y;
};

int x_min = -200, x_max = 200, y_min = -200, y_max = 200; // Clipping window bounds
vector<Point> polygonVertices;  // Store the vertices of the polygon
bool clippingDone = false;  // Flag to check if the polygon is clipped

// Define outcodes
#define INSIDE 0  // 0000
#define LEFT 1    // 0001
#define RIGHT 2   // 0010
#define BOTTOM 4  // 0100
#define TOP 8     // 1000

// Function to compute the outcode for a point
int computeOutCode(Point p) {
    int outcode = INSIDE;
    if (p.x < x_min) outcode |= LEFT;
    if (p.x > x_max) outcode |= RIGHT;
    if (p.y < y_min) outcode |= BOTTOM;
    if (p.y > y_max) outcode |= TOP;
    return outcode;
}

// Function to perform the Cohen-Sutherland clipping algorithm
void cohenSutherlandClip(vector<Point>& polygon) {
    vector<Point> clippedPolygon;

    for (size_t i = 0; i < polygon.size(); i++) {
        Point p1 = polygon[i];
        Point p2 = polygon[(i + 1) % polygon.size()];

        int outcode1 = computeOutCode(p1);
        int outcode2 = computeOutCode(p2);

        bool accept = false;

        while (true) {
            if ((outcode1 == 0) && (outcode2 == 0)) {  // Both points inside
                accept = true;
                clippedPolygon.push_back(p1);
                clippedPolygon.push_back(p2);
                break;
            } else if (outcode1 & outcode2) {  // Both points share an outside region
                break;
            } else {
                int outcodeOut;
                Point p;

                if (outcode1 != 0) outcodeOut = outcode1;
                else outcodeOut = outcode2;

                if (outcodeOut & TOP ) {
                    p.x = p1.x + (p2.x - p1.x) * (y_max - p1.y) / (p2.y - p1.y);
                    p.y = y_max;
                } else if (outcodeOut & BOTTOM ) {
                    p.x = p1.x + (p2.x - p1.x) * (y_min - p1.y) / (p2.y - p1.y);
                    p.y = y_min;
                } else if (outcodeOut & RIGHT ) {
                    p.y = p1.y + (p2.y - p1.y) * (x_max - p1.x) / (p2.x - p1.x);
                    p.x = x_max;
                } else if (outcodeOut & LEFT ) {
                    p.y = p1.y + (p2.y - p1.y) * (x_min - p1.x) / (p2.x - p1.x);
                    p.x = x_min;
                }

                if (outcodeOut == outcode1) {
                    p1 = p;
                    outcode1 = computeOutCode(p1);
                } else {
                    p2 = p;
                    outcode2 = computeOutCode(p2);
                }
            }
        }
    }

    polygon = clippedPolygon;
}

// Function to draw the clipping window
void drawClippingWindow() {
    glColor3f(0, 0, 0); // Black color
    glBegin(GL_LINE_LOOP);
    glVertex2i(x_min, y_min);
    glVertex2i(x_max, y_min);
    glVertex2i(x_max, y_max);
    glVertex2i(x_min, y_max);
    glEnd();
}

// Function to draw the polygon
void drawPolygon(vector<Point>& polygon) {
    if (polygon.size() < 2) return;

    glColor3f(0, 0, 1); // Blue color
    glBegin(GL_LINE_LOOP);
    for (const auto& p : polygon) {
        glVertex2i(p.x, p.y);
    }
    glEnd();
}

// Mouse click handler
void mouseClick(int button, int state, int x, int y) {
    if (button == GLUT_LEFT_BUTTON && state == GLUT_DOWN) {
        x = x - 300; // Center X
        y = 300 - y; // Flip Y

        polygonVertices.push_back({x, y});

        if (polygonVertices.size() >= 2) {
            glutPostRedisplay();
        }
    }
}

// Keyboard handler
void keyboard(unsigned char key, int x, int y) {
    if (key == 'c' || key == 'C') {
        cohenSutherlandClip(polygonVertices);
        clippingDone = true;

        glutPostRedisplay();
    }
}

// Initialization
void initOpenGL() {
    glClearColor(1, 1, 1, 1);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(-300, 300, -300, 300);
}

// ✅ Updated display function
void display() {
    glClear(GL_COLOR_BUFFER_BIT);
    drawClippingWindow();

    if (!polygonVertices.empty()) {
        drawPolygon(polygonVertices);  // Before or after clipping
    }

    glFlush();
}

// Main function
int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(600, 600);
    glutCreateWindow("Cohen-Sutherland Polygon Clipping");

    initOpenGL();

    glutDisplayFunc(display);
    glutMouseFunc(mouseClick);
    glutKeyboardFunc(keyboard);

    glutMainLoop();
    return 0;
}



