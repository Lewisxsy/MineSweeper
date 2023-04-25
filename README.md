本项目为c#实现的窗体应用程序-扫雷小游戏，为windows程序设计实验课程设计。

设计思路：
c# 扫雷游戏
1、UI界面绘制
2、埋雷
3、扫雷
 1）插旗
 2）爆炸
 3）无雷
  a）显示数字
  b）展开区域
4、计分

UI界面绘制：雷区、开始游戏、计时器、分数
  自定义雷区大小，设定好雷区每个按钮的大小参数，通过add函数生成雷区的所有按钮，地雷按钮生成时属性visible设置为false；
  计时器：label1，开始游戏后计时并显示时间；
  分数：label2，统计分数并显示；
  开始游戏：鼠标左键点击开始游戏按钮后触发事件，在雷区中随机生成设定数量的地雷，初始化游戏并重新生成地雷。
  鼠标左键点击雷区中的按钮后判定事件，若点击的按钮为地雷则游戏结束；若点击的按钮不为地雷且附近八格内有地雷，显示附近八格内的地雷数量；若此地雷数量为0，
  展开相邻所有地雷数量为0的按钮（为0迭代，其它数字或边界停止）。鼠标右键点击判定事件为插旗。
  判定胜利，计分并计入txt文件排行榜。

# MineSweeper
程序设计：扫雷
主窗体mainform代码设计实现：

```namespace WindowsFormsApp2
{
    public partial class 主窗体 : Form
    {
        //按钮组件
        Button[] btns;
        //记录每一个格子的数据(-1:地雷,-2:已被点击过
        private int[,] numbers;
        //地雷信息
        private bool[,] isBomb;
        //是否点击过
        private bool[,] isClicked;
        //是否被插旗
        private bool[,] isFlaged;
        //难度
        public int length;
        //地雷数量
        public int bombNumber;
        //插旗数量
        int flagNumber;
        //点击过的数量
        int clickNumber;
        //判断是否为第一次点击
        bool firstClick = true;
        int initialX;
        int initialY;
        //游戏花费的时间
        int startTime;
        int endTime;
        //炸弹图片路径
        public static string bombPath = "BombImage.png";
        //旗子图片路径
        public static string flagPath = "FlagImage.png";
        public 主窗体(int newBombNumber = 10, int newLength = 10)
        {
            //地雷数量
            bombNumber = newBombNumber;
            //新地图
            length = newLength;
            //加载组件
            InitializeComponent();
        }
        private void Form1_Load(object sender, EventArgs e)
        {
            numbers = new int[length, length];
            isBomb = new bool[length, length];
            isClicked = new bool[length, length];
            isFlaged = new bool[length, length];
            startTime = 0;
            endTime = 0;
            clickNumber = 0;
            this.Text = "扫雷(大小:" + length + ",雷数:" + bombNumber + ")";
            for (int i = 0; i < length; i++)
            {
                for (int j = 0; j < length; j++)
                {
                    numbers[i, j] = 0;
                    isBomb[i, j] = false;
                    isClicked[i, j] = false;
                    isFlaged[i, j] = false;
                }
            }
            btns = new Button[length * length];
            flagNumber = 0;
            int x0 = 5, y0 = 5, w = 30, d = w + 5;
            for (int i = 0; i < btns.Length; i++)
            {
                Button btn = new Button();
                int r = i / length;  //行
                int c = i % length;  //列
                btn.Left = x0 + c * d;
                btn.Top = y0 + r * d;
                btn.Width = w;
                btn.Height = w;
                btn.Font = new Font("楷体", 14);
                btn.Visible = true;
                btns[i] = btn;
                this.pnlBoard.Controls.Add(btn);
                btn.MouseDown += new System.Windows.Forms.MouseEventHandler(this.UpdateClick);
                btn.Name = i.ToString();
            }
        }
        protected void UpdateClick(object sender, MouseEventArgs e)
        {
            Button button = (Button)sender;
            int placeMouse = int.Parse(button.Name);
            int x = placeMouse / length;
            int y = placeMouse % length;
            if (e.Button == MouseButtons.Left)
            {
                //MessageBox.Show(button.Name+" ("+x+","+y+")");
                //等价于对button操作
                //判断是否为初次点击
                if (firstClick)
                {
                    initialX = x;
                    initialY = y;
                    //将该位置更改为有地雷再进行生成
                    isBomb[x, y] = true;
                    //随机生成地图
                    GenerateMap();
                    //这个位置不可能是地雷
                    isBomb[x, y] = false;
                    //不再是初次点击
                    firstClick = false;
                }
                //处理当前点击的按钮的信息
                //按钮已被点击过
                if (isClicked[x, y])
                    return;//直接返回
                //点击了一次
                clickNumber++;
                //如果该位置是地雷
                if (isBomb[x, y])
                {
                    GameOver();//游戏结束
                    return;
                }
                //其他情况
                //被点击过了
                isClicked[x, y] = true;
                //更改色调，以区分点击与未点击
                button.BackColor = Color.DarkGray;
                //显示地图
                if (numbers[x, y] != 0)
                    button.Text = numbers[x, y].ToString();
                spreadMap(x, y);
                //点击获胜
                if(clickNumber==length*length-bombNumber)
                {
                    GameWin();
                }
            }
            else if (e.Button == MouseButtons.Right)
            {
                //点击过了，就直接跳过
                if (isClicked[x, y] == true)
                    return;
                if(isFlaged[x,y]==false)
                {
                    // 显示炸弹图
                    button.BackgroundImage = Image.FromFile(flagPath);
                    //标记
                    isFlaged[x, y] = true;
                    //插旗数量++
                    flagNumber++;
                }
                else
                {
                    //撤回标记
                    button.BackgroundImage = null;
                    isFlaged[x, y] = false;
                    flagNumber--;
                }
                //判断是否能通过插旗获得游戏胜利
                if (flagNumber==bombNumber)
                {
                    for (int i = 0; i < length; i++)
                    {
                        for (int j = 0; j < length; j++)
                        {
                            if (isFlaged[i, j] == false)
                                continue;
                            if (isBomb[i, j] == false)
                                return;//插旗错误，无法获得游戏胜利
                        }
                    }
                    GameWin();
                }
            }
        }
        //随机生成地图
        public void GenerateMap()
        {
            Random r = new Random();
            int x, y;
            //随机产生length个地雷
            for (int i = 0; i < bombNumber; i++)
            {
                do
                {
                    //随机坐标
                    x = r.Next(length);
                    y = r.Next(length);
                }
                while (isBomb[x, y] == true);
                //该位置更改为地雷
                isBomb[x, y] = true;
                //左上
                if (x != 0 && y != 0)
                    numbers[x - 1, y - 1]++;
                //正上方
                if (x != 0)
                    numbers[x - 1, y]++;
                //右上方
                if (x != 0 && y != length - 1)
                    numbers[x - 1, y + 1]++;
                //正左方
                if (y != 0)
                    numbers[x, y - 1]++;
                //正右方
                if (y != length - 1)
                    numbers[x, y + 1]++;
                //左下方
                if (x != length - 1 && y != 0)
                    numbers[x + 1, y - 1]++;
                //正下方
                if (x != length - 1)
                    numbers[x + 1, y]++;
                //右下方
                if (x != length - 1 && y != length - 1)
                    numbers[x + 1, y + 1]++;
            }
        }
        public void GameOver()
        {
            for (int i = 0; i < length; i++)
            {
                for (int j = 0; j < length; j++)
                {
                    //如果之前被插了旗子
                    if (isFlaged[i, j] == true)
                    {
                        isFlaged[i, j] = false;
                        btns[i * length + j].BackgroundImage = null;
                    }
                    if (isClicked[i, j] == true)
                        continue;
                    //更改色调，以区分点击与未点击
                    btns[i*length+j].BackColor = Color.DarkGray;
                    //全部设置为完成点击
                    isClicked[i, j] = true;
                    if (isBomb[i, j])
                    {
                        btns[i * length + j].BackgroundImage = Image.FromFile(bombPath,true);
                        continue;
                    }
                    //显示地图
                    if (numbers[i, j] != 0)
                        btns[i * length + j].Text = numbers[i, j].ToString();
                }
            }
            MessageBox.Show("你输了！");
            RestartGame(bombNumber, length);
        }
        public void GameWin()
        {
            //显示地图全貌
            for (int i = 0; i < length; i++)
            {
                for (int j = 0; j < length; j++)
                {
                    //如果之前被插了旗子
                    if (isFlaged[i, j] == true)
                    {
                        isFlaged[i, j] = false;
                        btns[i * length + j].BackgroundImage = null;
                    }
                    if (isClicked[i, j] == true)
                        continue;
                    //更改色调，以区分点击与未点击
                    btns[i * length + j].BackColor = Color.DarkGray;
                    //全部设置为完成点击
                    isClicked[i, j] = true;
                    if (isBomb[i, j])
                    {
                        btns[i * length + j].BackgroundImage = Image.FromFile(bombPath, true);
                        continue;
                    }
                    //显示地图
                    if (numbers[i, j] != 0)
                        btns[i * length + j].Text = numbers[i, j].ToString();
                }
            }
            游戏胜利__ fp = new 游戏胜利__(length, bombNumber, endTime - startTime);
            fp.Owner = this;
            this.Hide();
            fp.ShowDialog();
        }
        //以x,y为起点扩展地图
        public void spreadMap(int x,int y)
        {
            //左上
            if (x != 0 && y != 0 && isBomb[x - 1, y - 1] == false && isClicked[x - 1, y - 1] == false)
            {
                //点击过了
                isClicked[x - 1, y - 1] = true;
                //更改色调，以区分点击与未点击
                btns[(x - 1) * length + y - 1].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x-1, y-1] == true)
                {
                    isFlaged[x-1, y-1] = false;
                    btns[(x - 1) * length + y - 1].BackgroundImage = null;
                }
                if (numbers[x - 1, y - 1] != 0)
                    btns[(x - 1) * length + y - 1].Text = numbers[x - 1, y - 1].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x - 1, y - 1);
                }
            }
            //正上
            if (x != 0 && isBomb[x - 1, y] == false && isClicked[x - 1, y ] == false)
            {
                //点击过了
                isClicked[x - 1, y] = true;
                //更改色调，以区分点击与未点击
                btns[(x - 1) * length + y].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x - 1, y ] == true)
                {
                    isFlaged[x - 1, y - 1] = false;
                    btns[(x - 1) * length + y ].BackgroundImage = null;
                }
                if (numbers[x - 1, y] != 0)
                    btns[(x - 1) * length + y].Text = numbers[x - 1, y].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x - 1, y);
                }
            }
            //右上
            if (x != 0 && y!=length-1&&isBomb[x - 1, y+1] == false && isClicked[x - 1, y + 1] == false)
            {
                //点击过了
                isClicked[x - 1, y + 1] = true;
                //更改色调，以区分点击与未点击
                btns[(x - 1) * length + y + 1].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x - 1, y + 1] == true)
                {
                    isFlaged[x - 1, y + 1] = false;
                    btns[(x - 1) * length + y + 1].BackgroundImage = null;
                }
                if (numbers[x - 1, y + 1] != 0)
                    btns[(x - 1) * length + y + 1].Text = numbers[x - 1, y + 1].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x - 1, y + 1);
                }
            }
            //正左方
            if (y != 0 && isBomb[x, y - 1] == false && isClicked[x , y - 1] == false)
            {
                //点击过了
                isClicked[x, y - 1] = true;
                //更改色调，以区分点击与未点击
                btns[x * length + y - 1].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x , y - 1] == true)
                {
                    isFlaged[x , y - 1] = false;
                    btns[x * length + y - 1].BackgroundImage = null;
                }
                if (numbers[x, y - 1] != 0)
                    btns[x * length + y - 1].Text = numbers[x, y - 1].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x, y - 1);
                }
            }
            //正右方
            if (y != length - 1 && isBomb[x, y + 1] == false && isClicked[x, y + 1] == false)
            {
                //点击过了
                isClicked[x, y + 1] = true;
                //更改色调，以区分点击与未点击
                btns[x * length + y + 1].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x, y + 1] == true)
                {
                    isFlaged[x, y + 1] = false;
                    btns[x * length + y + 1].BackgroundImage = null;
                }
                if (numbers[x, y + 1] != 0)
                    btns[x * length + y + 1].Text = numbers[x, y + 1].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x, y + 1);
                }
            }
            //左下方
            if (x != length - 1 && y != 0 && isBomb[x + 1, y - 1] == false && isClicked[x + 1, y - 1] == false)
            {
                //点击过了
                isClicked[x + 1, y - 1] = true;
                //更改色调，以区分点击与未点击
                btns[(x + 1) * length + y - 1].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x + 1, y - 1] == true)
                {
                    isFlaged[x + 1, y - 1] = false;
                    btns[(x + 1) * length + y - 1].BackgroundImage = null;
                }
                if (numbers[x + 1, y - 1] != 0)
                    btns[(x + 1) * length + y - 1].Text = numbers[x + 1, y - 1].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x + 1, y - 1);
                }
            }
            //正下方
            if (x != length - 1 && isBomb[x + 1, y] == false && isClicked[x + 1, y] == false)
            {
                //点击过了
                isClicked[x + 1, y] = true;
                //更改色调，以区分点击与未点击
                btns[(x + 1) * length + y].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x + 1, y ] == true)
                {
                    isFlaged[x + 1, y ] = false;
                    btns[(x + 1) * length + y ].BackgroundImage = null;
                }
                if (numbers[x + 1, y] != 0)
                    btns[(x + 1) * length + y].Text = numbers[x + 1, y].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x + 1, y);
                }
            }
            //右下方
            if (x != length - 1 && y != length - 1 && isBomb[x + 1, y + 1] == false && isClicked[x + 1, y + 1] == false)
            {
                //点击过了
                isClicked[x + 1, y + 1] = true;
                //更改色调，以区分点击与未点击
                btns[(x + 1) * length + y + 1].BackColor = Color.DarkGray;
                //点击次数
                clickNumber++;
                if (isFlaged[x + 1, y + 1] == true)
                {
                    isFlaged[x + 1, y + 1] = false;
                    btns[(x + 1) * length + y + 1].BackgroundImage = null;
                }
                if (numbers[x + 1, y + 1] != 0)
                    btns[(x + 1) * length + y + 1].Text = numbers[x + 1, y + 1].ToString();
                else
                {
                    //非零需要继续拓展
                    spreadMap(x + 1, y + 1);
                }
            }
            //返回
            return;
        }
        private void 新游戏ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            //MessageBox.Show()
            RestartGame(bombNumber, length);
        }
        public void RestartGame(int newBombNumber,int newLength)
        {
            this.Hide();   //先隐藏主窗体
            主窗体 form1 = new 主窗体(newBombNumber, newLength); //重新实例化此窗体
            form1.Height = 35 * newLength + 80;
            form1.Width = 35 * newLength + 60;
            form1.ShowDialog();//已模式窗体的方法重新打开
            this.Close();
        }
        private void 炸弹数目ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            //此时主窗体禁止修改
            this.Hide();
            炸弹数量设置 fp = new 炸弹数量设置();
            fp.Owner = this;
            fp.ShowDialog();
            
        }
        private void 排行榜ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            排行榜 fp = new 排行榜();
            this.Hide();
            fp.Owner = this;
            fp.ShowDialog();
        }
        private void 地图大小ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            //此时主窗体禁止修改
            this.Hide();
            地图大小设置 fp = new 地图大小设置();
            fp.Owner = this;
            fp.ShowDialog();
        }
        private void pnlBoard_Paint(object sender, PaintEventArgs e)
        {

        }
    }
}

启动类program设计代码实现：
namespace WindowsFormsApp2
{
    static class Program
    {
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            Application.Run(new 主窗体());
        }
    }
}

得分类score设计代码实现：
namespace WindowsFormsApp2
{
    class Score
    {
        public string name;
        public int length;
        public int bombNumber;
        //构造函数
        public Score(string n,int l,int b)
        {
            this.name = n;
            this.length = l;
            this.bombNumber = b;
        }
    }
}

地图大小设置窗体设计实现：
namespace WindowsFormsApp2
{
    public partial class 地图大小设置 : Form
    {
        public 地图大小设置()
        {
            InitializeComponent();
        }

        private void button1_Click(object sender, EventArgs e)
        {
            int number = int.Parse(numericUpDown1.Value.ToString());
            主窗体 fp = (主窗体)this.Owner;
            int bombNumber = fp.bombNumber;
            if (number*number <= bombNumber)
            {
                MessageBox.Show("地图过小，无法适应炸弹数量！");
                return;
            }
            //fp.Show();
            this.Hide();
            fp.RestartGame(bombNumber, number);
        }

        private void 地图大小设置_FormClosed(object sender, FormClosedEventArgs e)
        {
            try { this.Owner.Show(); }
            catch { }
        }

        private void 地图大小设置_Load(object sender, EventArgs e)
        {

        }
    }
}

排行榜窗体设计实现：
namespace WindowsFormsApp2
{
    public partial class 排行榜 : Form
    {
        Score[] persons;
        public 排行榜()
        {
            InitializeComponent();
        }
        private void 排行榜_FormClosed(object sender, FormClosedEventArgs e)
        {
            this.Owner.Show();
        }
        private void 排行榜_Load(object sender, EventArgs e)
        {
            StreamReader fp = new StreamReader("Achievements.txt");
            char[] deli = { ',', ' ', '\t' };
            int count = 0;
            while (fp.Peek()>0)
            {
                //只显示前十名
                //if (count >= 10)
                //   return;
                //string lines = fp.ReadLine();
                //string[] unit = lines.Split(deli, StringSplitOptions.RemoveEmptyEntries);
                fp.ReadLine();
                count++;
            }
            fp.Close();
            persons = new Score[count];
            //重新加载
            fp = new StreamReader("Achievements.txt");
            int temp = 0;
            while (fp.Peek() > 0)
            {
                //只显示前十名
                //if (count >= 10)
                //   return;
                string lines = fp.ReadLine();
                string[] unit = lines.Split(deli, StringSplitOptions.RemoveEmptyEntries);
                persons[temp] = new Score(unit[0], int.Parse(unit[1]), int.Parse(unit[2]));
                temp++;
            }
            fp.Close();
            sort(count);
            //显示数据
            for(int i=0;i<count;i++)
            {
                listBox1.Items.Add(persons[i].name);
                listBox2.Items.Add(persons[i].length+"X"+persons[i].length);
                listBox3.Items.Add(persons[i].bombNumber);
            }
        }
        //排序
        public void sort(int length)
        {
            //冒泡排序
            for (int i = length - 1; i >= 0; i--)
            {
                for (int j = 0; j < i; j++)
                {
                    //bombNumber/(length*length)反映了炸弹密度，即难度
                    //使用乘法，避免浮点数运算
                    if (persons[j].bombNumber*persons[i].length * persons[i].length < persons[i].bombNumber * persons[j].length * persons[j].length)
                    {
                        Score s = persons[i];
                        persons[i] = persons[j];
                        persons[j] = s;
                    }
                }
            }
        }
    }
}


游戏胜利类窗体设计实现：
namespace WindowsFormsApp2
{
    public partial class 游戏胜利__ : Form
    {
        int length;
        int bombNumber;
        int useTime;
        public 游戏胜利__(int l,int b,int u)
        {
            length = l;
            bombNumber = b;
            useTime = u;
            InitializeComponent();
        }

        private void button1_Click(object sender, EventArgs e)
        {
            string names = textBox1.Text;
            if(names.Length==0)
            {
                MessageBox.Show("君の名は！！");
                return;
            }
            FileStream fp2 = new FileStream("Achievements.txt", FileMode.Append);
            StreamWriter fp = new StreamWriter(fp2);
            fp.Write(names + " " + length + " " + bombNumber + " " + useTime + "\r\n");
            fp.Close();
            主窗体 fps = (主窗体)this.Owner;
            //this.Close();
            this.Hide();
            fps.RestartGame(bombNumber, length);
        }

        private void 游戏胜利___FormClosed(object sender, FormClosedEventArgs e)
        {
            try { this.Owner.Show(); }
            catch { }
        }

        private void 游戏胜利___Load(object sender, EventArgs e)
        {

        }
    }
}

炸弹数量设置类窗体设计实现：
namespace WindowsFormsApp2
{
    public partial class 炸弹数量设置 : Form
    {
        public 炸弹数量设置()
        {
            InitializeComponent();
        }
        private void button1_Click(object sender, EventArgs e)
        {
            int number = int.Parse(numericUpDown1.Value.ToString());
            主窗体 fp = (主窗体)this.Owner;
            int length = fp.length;
            if (number >= length * length)
            {
                MessageBox.Show("炸弹数量过多，请重新设置！");
                return;
            }
            //fp.Show();
            this.Hide();
            fp.RestartGame(number, length);
            
        }
        private void 炸弹数量设置_Load(object sender, EventArgs e)
        {

        }
        private void 炸弹数量设置_FormClosed(object sender, FormClosedEventArgs e)
        {
            ////this.Close();
            try { this.Owner.Show(); }
            catch { }
        }
        private void numericUpDown1_ValueChanged(object sender, EventArgs e)
        {

        }
        private void label1_Click(object sender, EventArgs e)
        {

        }
    }
}```

实验结果：
