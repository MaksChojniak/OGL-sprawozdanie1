pobierz plik [main.cpp](./main.cpp) 
lub skopiuj tekst poniżej:
<pre>
#include <GLFW/glfw3.h> 
#include<iostream> 
#include<fstream>
#include<stdlib.h>

using namespace std;

const int w = 480;
const int k = 640;
const int ps = 1;
int Rs[w][k];
int Gs[w][k];
int Bs[w][k];

int Rc[w][k];
int Gc[w][k];
int Bc[w][k];

int lk;
int lw;

int brightness = 0;
int brightness_step = 5;

const int kernelSize = 3;

int Mdp[kernelSize][kernelSize] = { 
    {1,1,1}, 
    {1,1,1}, 
    {1,1,1} 
};
//int sumMdp = 0;

int Mgp[kernelSize][kernelSize] = { 
    {-1,-1,-1}, 
    {-1, 9,-1}, 
    {-1,-1,-1} 
};
//int sumMgp = 0;

int Mkrawedz[kernelSize][kernelSize] = {
    {0,1,0},
    {1,-4,1},
    {0,1,0}
};

int Memboss[kernelSize][kernelSize] = {
    {-2,-1,0},
    {-1, 1,1},
    { 0, 1,2}
};



void display(float size) {

    glPushMatrix();
    glTranslated(0, 0, -6);
    glBegin(GL_POINTS);
    for (int i = 0; i < w; ++i)
    {
        for (int j = 0; j < lk; ++j)
        {
            glPointSize(ps);
            glColor3d((Rc[i][j] + brightness) / 255.0, (Gc[i][j] + brightness) / 255.0, (Bc[i][j] + brightness) / 255.0);
            glVertex3f(j, i, 0);
        }
    }
    glEnd();
    glPopMatrix();
}

const GLfloat light_ambient[] = { 0.0f, 0.0f, 0.0f, 1.0f };
const GLfloat light_diffuse[] = { 1.0f, 1.0f, 1.0f, 1.0f };
const GLfloat light_specular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
const GLfloat light_position[] = { 2.0f, 5.0f, 5.0f, 0.0f };

const GLfloat mat_ambient[] = { 0.7f, 0.7f, 0.7f, 1.0f };
const GLfloat mat_diffuse[] = { 0.8f, 0.8f, 0.8f, 1.0f };
const GLfloat mat_specular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
const GLfloat high_shininess[] = { 100.0f };


void zastosuj_maske(int m[kernelSize][kernelSize]) {
    int sum = 0;
    for (int s = 0; s < kernelSize; ++s)
    {
        for (int d = 0; d < kernelSize; ++d)
        {
            sum += m[s][d];
        }
    }
    if (sum == 0)
        sum = 1;

    for (int i = 1; i < w - 1; ++i)
    {
        for (int j = 1; j < lk - 1; ++j)
        {
            int pomR = 0;
            int pomG = 0;
            int pomB = 0;

            for (int s = 0; s < kernelSize; ++s)
            {
                for (int d = 0; d < kernelSize; ++d)
                {
                    pomR = pomR + (m[s][d] * Rs[i - 1 + s][j - 1 + d]);
                    pomG = pomG + (m[s][d] * Gs[i - 1 + s][j - 1 + d]);
                    pomB = pomB + (m[s][d] * Bs[i - 1 + s][j - 1 + d]);
                }
            }
            Rc[i][j] = pomR / sum;
            Gc[i][j] = pomG / sum;
            Bc[i][j] = pomB / sum;
        }
    }
}

int min(int a, int b) {
    return std::min(a,b);
}

int max(int a, int b) {
    return std::max(a, b);
}

void zastosuj_filtr(int (*fun)(int, int), int init) {


    for (int i = 1; i < w - 1; ++i)
    {
        for (int j = 1; j < lk - 1; ++j)
        {
            int maxR = init;
            int maxG = init;
            int maxB = init;
            for (int s = 0; s < kernelSize; ++s)
            {
                for (int d = 0; d < kernelSize; ++d)
                {
                    maxR = fun(maxR, Rs[i - 1 + s][j - 1 + d]);
                    maxG = fun(maxG, Gs[i - 1 + s][j - 1 + d]);
                    maxB = fun(maxB, Bs[i - 1 + s][j - 1 + d]);
                }
            }
            Rc[i][j] = maxR;
            Gc[i][j] = maxG;
            Bc[i][j] = maxB;
        }
    }
}


void key_callback(GLFWwindow* window, int key, int scancode, int action, int mods) {
    if (action == GLFW_PRESS) {
        switch (key) {
        case GLFW_KEY_ESCAPE:
            glfwSetWindowShouldClose(window, GLFW_TRUE);
            break;
        case GLFW_KEY_UP:
            brightness += brightness_step;
            break;
        case GLFW_KEY_DOWN:
            brightness -= brightness_step;
            break;
        case GLFW_KEY_LEFT:
            if (brightness_step > 1)
                brightness_step -= 1;
            cout << "brightness step: " << brightness_step << endl;
            break;
        case GLFW_KEY_RIGHT:
            brightness_step += 1;
            cout << "brightness step: " << brightness_step << endl;
            break;
        case GLFW_KEY_R:
            for (int i = 0; i < w; ++i)
            {
                for (int j = 0; j < lk; ++j)
                {
                    Rc[i][j] = Rs[i][j];
                    Gc[i][j] = 0;
                    Bc[i][j] = 0;
                }
            }
            break;
        case GLFW_KEY_T:
            for (int i = 0; i < w; ++i)
            {
                for (int j = 0; j < lk; ++j)
                {
                    Rc[i][j] = 0;
                    Gc[i][j] = Gs[i][j];
                    Bc[i][j] = 0;
                }
            }
            break;
        case GLFW_KEY_Y:
            for (int i = 0; i < w; ++i)
            {
                for (int j = 0; j < lk; ++j)
                {
                    Rc[i][j] = 0;
                    Gc[i][j] = 0;
                    Bc[i][j] = Bs[i][j];
                }
            }
            break;
        case GLFW_KEY_D:
            zastosuj_maske(Mdp);
            break;
        case GLFW_KEY_C:
            zastosuj_maske(Mgp);
            break;
        case GLFW_KEY_X:
            zastosuj_maske(Mkrawedz);
            break;
        case GLFW_KEY_Z:
            zastosuj_maske(Memboss);
            break;
        case GLFW_KEY_F:
            zastosuj_filtr(max, INT32_MIN);
            break;
        case GLFW_KEY_G:
            zastosuj_filtr(min, INT32_MAX);
            break;
        case GLFW_KEY_BACKSPACE:

            brightness = 0;

            for (int i = 0; i < w; ++i)
            {
                for (int j = 0; j < lk; ++j)
                {
                    Rc[i][j] = Rs[i][j];
                    Gc[i][j] = Gs[i][j];
                    Bc[i][j] = Bs[i][j];
                }
            }

            break;
        }
    }
}


int main(void)
{
    ifstream plik("C:\\Users\\Maks\\Programming\\learning\\politechnika-czestochowska\\grafika-komputerowa\\obraz4.txt");
    plik >> lw >> lk;
    cout << "wiersze=" << w << " kolumny=" << k << endl;
    for (int i = 0; i < w; ++i)
    {
        for (int j = 0; j < lk; ++j)
        {
            plik >> Rs[i][j];
            plik >> Gs[i][j];
            plik >> Bs[i][j];

            Rc[i][j] = Rs[i][j];
            Gc[i][j] = Gs[i][j];
            Bc[i][j] = Bs[i][j];
        }
    }
    plik.close();

    GLFWwindow* window;

    if (!glfwInit())
        return -1;

    //window = glfwCreateWindow(800, 600, "Przykladowe Okno GLFW", NULL, NULL);
    window = glfwCreateWindow(lk, lw, "Przykladowe Okno GLFW", NULL, NULL);

    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    glfwSetKeyCallback(window, key_callback);

    glfwMakeContextCurrent(window);

    while (!glfwWindowShouldClose(window))
    {
        int width, height;
        glfwGetFramebufferSize(window, &width, &height);
        glViewport(0, 0, width, height);
        glClear(GL_COLOR_BUFFER_BIT);

        glMatrixMode(GL_PROJECTION);
        glLoadIdentity();
        //glOrtho(-width / 2.0, width / 2.0, -height / 2.0, height / 2.0, -10.0, 10.0);
        glOrtho(0, lk, 0, lw, 2.0, 100.0);

        glMatrixMode(GL_MODELVIEW);

        float squareSize = static_cast<float>(std::min(width, height)) * 0.2f;

        display(squareSize);

        glEnable(GL_CULL_FACE);
        glClearColor(1, 1, 1, 1);
        glfwSwapBuffers(window);

        glEnable(GL_LIGHT0);
        glEnable(GL_NORMALIZE);
        glEnable(GL_COLOR_MATERIAL);
        glEnable(GL_LIGHTING);

        glLightfv(GL_LIGHT0, GL_AMBIENT, light_ambient);
        glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse);
        glLightfv(GL_LIGHT0, GL_SPECULAR, light_specular);
        glLightfv(GL_LIGHT0, GL_POSITION, light_position);

        glMaterialfv(GL_FRONT, GL_AMBIENT, mat_ambient);
        glMaterialfv(GL_FRONT, GL_DIFFUSE, mat_diffuse);
        glMaterialfv(GL_FRONT, GL_SPECULAR, mat_specular);
        glMaterialfv(GL_FRONT, GL_SHININESS, high_shininess);

        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}

</pre>
