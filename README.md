# cg

#include <stdio.h>
#include <GL/glut.h>

#define W 40 //地面の幅
#define D 40 //地面の長さ

//テクスチャ
//芝
#define TEXWIDTH1 256 //幅
#define TEXHEIGHT1 256 //高さ
GLuint texid_1;
static const char texture1[] = "shibahu.png";

//地面
static void Ground(double height){
const static GLfloat ground[][4] =
  {{0.6, 0.6, 0.6, 1.0},
   {0.3, 0.3, 0.3, 1.0}};

 int i,j;
 glEnable(GL_ALPHA_TEST);
 glEnable(GL_TEXTURE_2D);
 glBindTexture(GL_TEXTURE_2D, texid_1);
 glBegin(GL_QUADS);
 glNormal3d(0.0, 1.0, 0.0);
 for(j=-D/2; j<D/2; ++j){
   for(i=-W/2; i<W/2; ++i){
     //glMaterialfv(GL_FRONT, GL_DIFFUSE, ground[(i+j)&1]);
     glVertex3d((GLdouble)i, height, (GLdouble)j);
     glVertex3d((GLdouble)i, height, (GLdouble)(j+1));
     glVertex3d((GLdouble)(i+1), height, (GLdouble)(j+1));
     glVertex3d((GLdouble)(i+1), height, (GLdouble)j);
   }
 }
 glEnd();
 glBindTexture(GL_TEXTURE_2D, texid_1);
}

//画面表示
static void display(void){
  //光源の位置
  const static GLfloat lightpos[] = {3.0, 4.0, 5.0, 0.0};

  //画面クリア
  glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

  //モデルビュー変換行列の初期化
  glLoadIdentity();

  //光源の位置を設定
  glLightfv(GL_LIGHT0, GL_POSITION, lightpos);

  //視点の移動
  //glTranslated(0.0, -1.5, -10);

  //シーンの描画
  Ground(0.0);

  glFlush();
}

static void resize(int w, int h){
  //ウィンドウ全体をビューポートにする
  glViewport(0, 0, w, h);

  //透視変換行列の指定
  glMatrixMode(GL_PROJECTION);

  //透視変換行列の初期化
  glLoadIdentity();
  gluPerspective(30.0, (double)w/(double)h, 1.0, 100.0);
  
  //視点移動
  gluLookAt(0.0, 20.0, 60.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0);

  //モデルビュー変換行列の指定
  glMatrixMode(GL_MODELVIEW);
}

static void keyboard(unsigned char key, int x, int y){
  //ESCかqをタイプしたら終了
  if(key == '\033' || key == 'q'){
    exit(0);
  }
}

static void init(void){
  //テクスチャの読み込み
  GLubyte texture_buf1[TEXHEIGHT1][TEXWIDTH1][4];
  FILE *fp;

  //1枚目
  if((fp = fopen(texture1, "rb")) != NULL){
    fread(texture_buf1, sizeof texture_buf1, 1, fp);
    fclose(fp);
  }
  else{
    perror(texture1);
  }

  glGenTextures(1, &texid_1); //ID作成
  glBindTexture(GL_TEXTURE_2D, texid_1);

  glPixelStorei(GL_UNPACK_ALIGNMENT, 4);

  //テクスチャの割り当て
  gluBuild2DMipmaps(GL_TEXTURE_2D, GL_RGBA, TEXWIDTH1, TEXHEIGHT1,
		    GL_RGBA, GL_UNSIGNED_BYTE, texture_buf1);
  //拡大縮小の指定
  glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
  glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);

  //環境
  glTexEnvi(GL_TEXTURE_ENV, GL_TEXTURE_ENV_MODE, GL_MODULATE);

  glBindTexture(GL_TEXTURE_2D, 0);
  
#if 0
  /* 混合する色の設定 */
  static const GLfloat blend[] = { 0.0, 1.0, 0.0, 1.0 };
  glTexEnvfv(GL_TEXTURE_ENV, GL_TEXTURE_ENV_COLOR, blend);
#endif

  glAlphaFunc(GL_GREATER, 0.5);

  //初期設定
  glClearColor(1.0, 1.0, 1.0, 1.0);
  glEnable(GL_DEPTH_TEST);
  glEnable(GL_CULL_FACE);
  glEnable(GL_LIGHTING);
  glEnable(GL_LIGHT0);
}

int main(int argc, char *argv[]){
  glutInitWindowPosition(100, 100);  //ウィンドウの位置
  glutInitWindowSize(640, 480);  //ウィンドウのサイズ
  glutInit(&argc, argv);
  glutInitDisplayMode(GLUT_RGBA | GLUT_DEPTH);
  glutCreateWindow(argv[0]);
  glutDisplayFunc(display);
  glutReshapeFunc(resize);
  glutKeyboardFunc(keyboard);
  init();
  glutMainLoop();
  return 0;
}
