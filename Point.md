import java.awt.Color;
public class Point {
	private int x;
	//棋盘中X的索引
	private int y;
	//棋盘中Y的索引
	private Color color;//颜色
	public static final int DIAMETER = 30;
	//直径
	public Point(int x,int y,Color color){
		this.x=x;
		this.y=y;
		this.color =color;
	}
	//拿到棋盘中的Y索引
	public int getX(){
		 return x;
	}
	
	public int getY(){
		 return y;
	}
	
	//得到颜色
	public Color getColor(){
		return color;
	}
}


import javax.swing.*;
import java.awt.*;
import java.awt.event.MouseListener;
import java.awt.event.MouseMotionListener;
import java.awt.event.MouseEvent;
/*
 五子棋-棋盘类
 */
public class ChessBoard extends JPanel implements MouseListener {
	public static final int MARGIN = 30;
	//边距
	protected static final int GRID_SPAN = 35;
	//网格间距
	public static final int ROWS = 10;
	//棋盘行数
	public static final int COLS = 10;
	
	//棋盘列数
	Point[] chessList = new Point[(ROWS+1)*(COLS+1)];
	//初始化每个数组元素为null
	boolean isBlack = true;
	//默认开始是黑棋先下
	boolean gameOver = false;
	//游戏是否结束
	int chessCount;
	//当前棋盘棋子的个数
	int xIndex,yIndex;
	//当前刚下的棋子的索引
	public ChessBoard(){
		super.setBackground(Color.ORANGE);
		//setBackground(Color.ORANGE);//设置背景颜色为橘黄色
		addMouseListener(this);//添加监听器
		addMouseMotionListener(new MouseMotionListener(){//匿名内部类
			public void mouseDragged(MouseEvent e){	
			}
			public void mouseMoved(MouseEvent e){
				int x1 = (e.getX()- MARGIN +GRID_SPAN/2)/GRID_SPAN;
				//将鼠标单击的坐标位置转换成网格索引
				int y1 = (e.getY()- MARGIN +GRID_SPAN/2)/GRID_SPAN;
				//游戏已经结束，落在棋盘外，x、y位置已经有棋子存在，不能下
				if(x1<0||x1>ROWS||y1<0||y1>COLS||gameOver||findChess(x1,y1))
					setCursor(new Cursor(Cursor.DEFAULT_CURSOR));
					//设置成默认形状
				else
					setCursor(new Cursor(Cursor.HAND_CURSOR));
				//设置成手型
			}
		});
	}
	


	//绘制
	public void paintComponent(Graphics g){
		super.paintComponent(g);
		//画棋类
		for(int i=0;i<=ROWS;i++){//画横线
			g.drawLine(MARGIN,MARGIN+i*GRID_SPAN,MARGIN+COLS*GRID_SPAN,MARGIN+i*GRID_SPAN);	
		}
		for(int i=0;i<=COLS;i++){//画直线
			g.drawLine(MARGIN+i*GRID_SPAN,MARGIN,MARGIN+i*GRID_SPAN,MARGIN+ROWS*GRID_SPAN);
		}
		//画棋子
		for(int i=0;i<chessCount;i++) {
			int xPos = chessList[i].getX()*GRID_SPAN+MARGIN;
			//网络交叉点的x坐标
			int yPos = chessList[i].getY()*GRID_SPAN+MARGIN;
			//网络交叉点的y坐标
			g.setColor(chessList[i].getColor());//设置颜色
			g.fillOval(xPos-Point.DIAMETER/2, yPos-Point.DIAMETER/2, Point.DIAMETER, Point.DIAMETER);
			//标记最后一个棋子的红矩形框
			if(i == chessCount -1){
				//最后一个棋子
				g.setColor(Color.red);
				g.drawRect(xPos - Point.DIAMETER/2,yPos-Point.DIAMETER/2,Point.DIAMETER,Point.DIAMETER);
			}
		}
		
		
		}
	
	//鼠标按键在组建上按下时调用
	public void mousePressed(MouseEvent e){
		//游戏已经结束，不能下
		if(gameOver)
			return;
	String colorName = isBlack ?"黑棋":"白棋";
	xIndex = (e.getX() - MARGIN + GRID_SPAN/2)/GRID_SPAN;
	//将鼠标单击的坐标位置转换成网格索引
	yIndex = (e.getY() - MARGIN + GRID_SPAN/2)/GRID_SPAN;
	//落在棋盘外，不能下
	if(xIndex <0 || xIndex > ROWS || yIndex < 0 || yIndex > COLS)
		return;
	//x、y位置都已经有棋子存在，不能下
	if(findChess(xIndex , yIndex))
		return;
	
	Point ch = new Point(xIndex,yIndex,isBlack ? Color.black:Color.white);
	chessList[chessCount++] = ch;
	repaint();//通知系统重新绘制
	if(isWin()){
		//给出胜利信息，不能再继续下棋
		String msg = String.format("恭喜,%s赢了！", colorName);
		JOptionPane.showMessageDialog(this, msg);
		gameOver = true;
	}
		isBlack = !isBlack;	
	}
	//覆盖MouseListener的方法
	public void mouseClicked(MouseEvent e){
	}//鼠标按键在组件上单击（按下并释放）时调用
	public void mouseEntered(MouseEvent e){
	}//鼠标进入到组件上时调用
	public void mouseExited(MouseEvent e){
	}//鼠标离开组件时调用
	public void mouseReleased(MouseEvent e){
	}//鼠标离开组件时调用
	//在棋子数组中查找是否有索引为x、y的棋子存在
	private boolean findChess(int x, int y){
		for(Point c : chessList){
			if(c !=null && c.getX() == x && c.getY() == y)
				return true;	
		}
				return false;
	}
	
		//判定哪方赢
	private boolean isWin(){
		int continueCount =1;
		//连续棋子的个数
		//横向向西寻找
		for(int x = xIndex-1; x>=0;x--){
			Color c = isBlack ? Color.black : Color.white;
			if(getChess(x,yIndex,c) !=null){
				continueCount++;	
			}else
				break;
		}
		//横向向东寻找
		for(int x  =xIndex+1;x<=ROWS;x++){
			Color c = isBlack ? Color.black : Color.white;
			if(getChess(x,yIndex,c) !=null){
				 continueCount++;
			}else 
				break;
		}
		//判定记录数大于等于五，即表示此方取胜
		if(continueCount>=5){
			return true;
		}else
			continueCount =1;
		//继续另一种情况的搜索：纵向
		//纵向向上寻找
			for(int y = yIndex - 1; y >= 0; y--){
				Color c =isBlack ? Color.black : Color.white;
				if(getChess(xIndex,y,c) !=null){
					continueCount++;
				}else
					break;
			}
		//纵向向下寻找
		for(int y = yIndex + 1; y<=ROWS; y++){
			Color c = isBlack ? Color.black : Color.white;
			if(getChess(xIndex,y,c)!=null){
				continueCount++;
			}else
				break;
		}
	
		if(continueCount>=5){
			return true;
		}else
			continueCount =1;
		
		//继续另一种情况的搜索：斜向
		//东北寻找
		for(int x = xIndex + 1,y=yIndex-1; y>=0 && x<=COLS; x++,y--){
			Color c = isBlack ? Color.black : Color.white;
			if(getChess(x,y,c)!=null){
				continueCount++;
			}else
				break;
			}
		
		//西南寻找
		for(int x = xIndex - 1,y=yIndex+1; y<=ROWS && x>=0; x--,y++){
			Color c = isBlack ? Color.black : Color.white;
			if(getChess(x,y,c)!=null){
				continueCount++;
			}else
				break;
		}
		if(continueCount >=5){
			return true;
		}else
			continueCount = 1;
		//继续另一种情况的搜索：斜向
		//西北寻找
		for(int x = xIndex - 1,y = yIndex-1; y>=0 && x>=0; x--,y--){
			Color c = isBlack ? Color.black : Color.white;
			if(getChess(x,y,c)!=null){
				continueCount++;
			}else
				break;
		
	}
	//西南寻找
		for(int x = xIndex + 1,y=yIndex+1; y<=ROWS && x<=COLS; x++,y++){
			Color c = isBlack ? Color.black : Color.white;
			if(getChess(x,y,c)!=null){
				continueCount++;
			}else
				break;
		}
		if(continueCount >=5){
			return true;
		}else
			continueCount = 1;
		return false;
	}
	private Point getChess(int xIndex, int yIndex, Color color){
		for(Point c: chessList){
			if(c !=null && c.getX() == xIndex && c.getY() == yIndex && c.getColor() == color)
				return c;
		}
		return null;
	}
	public void restartGame(){
		//清除棋子
		for(int i=0; i< chessList.length; i++)
			chessList[i]=null;
		//恢复游戏相关的变量值
		isBlack = true;
		gameOver = false;//游戏是否结束
		chessCount = 0;//当前棋盘的棋子个数
		//System.out.println(this.toString());
		//repaint();
			
	}
	//悔棋
	public void goback(){
		if (chessCount == 0)
			return;
		
		chessList[chessCount-1]=null;
		chessCount--;
		if(chessCount >0){
			xIndex = chessList[chessCount -1].getX();
			yIndex = chessList[chessCount -1].getY();
		}
		isBlack = !isBlack;
		//repaint();
	}
	//Dimension:矩形
	public Dimension getPreferredSize(){
		return new Dimension (MARGIN*2 + GRID_SPAN*COLS,MARGIN*2 + GRID_SPAN*ROWS);
	

	}
}	
	
	


import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.FlowLayout;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JMenu;
import javax.swing.JMenuBar;
import javax.swing.JMenuItem;
import javax.swing.JPanel;
public class StartChessJFrame  extends JFrame {
	private ChessBoard chessBoard;
	//对战面板
	private JPanel toolbar;
	//工具条面板
	private JButton startButton;
	private JButton backButton;
	private JButton exitButton;
	//“重新开始”按钮，“悔棋”按钮，“退出”按钮
	private JMenuBar menuBar;
	//菜单栏
	private JMenu sysMenu;
	//系统菜单
	private JMenuItem startMenuItem;
	private JMenuItem exitMenuIatem;
	private JMenuItem backMenuItem;
	//“重新开始”，“退出”和“悔棋”菜单项
	public StartChessJFrame(){
		setTitle("单机版五子棋");
		//设置标题
		chessBoard =new ChessBoard();
		//初始化面板对象
		//创建和添加菜单
		menuBar = new JMenuBar();
		//初始化菜单栏
		sysMenu = new JMenu("系统");
		//初始化菜单
		startMenuItem = new JMenuItem("重新开始");
		exitMenuIatem = new JMenuItem("退出");
		backMenuItem = new JMenuItem("悔棋");
		//初始化菜单项
		sysMenu.add(startMenuItem);
		//将三个菜单项添加到菜单上
		sysMenu.add(backMenuItem);
		sysMenu.add(exitMenuIatem);
		MyItemListener lis = new MyItemListener();
		//初始化按钮事件监听器内部类
		this.startMenuItem.addActionListener(lis);
		//将三个菜单项注册到事件监听器上
		backMenuItem.addActionListener(lis);
		exitMenuIatem.addActionListener(lis);
		menuBar.add(sysMenu);
		//将“系统”菜单添加到菜单栏上
		setJMenuBar(menuBar);
		//将menuBar设置为菜单栏
		toolbar =new JPanel();
		//工具面板实例化
		startButton = new JButton("重新开始");
		//三个按钮初始化
		backButton = new JButton("悔棋");
		exitButton = new JButton("退出");
		toolbar.setLayout(new FlowLayout(FlowLayout.LEFT));
		//将工具面板按钮用FlowLayout布局
		toolbar.add(startButton);
		//将三个按钮添加到工具面板上
		toolbar.add(backButton);
		toolbar.add(exitButton);
		startButton.addActionListener(lis);
		//将三个按钮注册监听事件
		backButton.addActionListener(lis);
		exitButton.addActionListener(lis);
		add(toolbar,BorderLayout.SOUTH);
		//将工具面板布局到界面“南”方也就是下面
		add(chessBoard);//将面板对象添加到窗体上
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		//设置界面关闭事件
		//setSize(600,650);
		pack();
		//自适应大小	

	}

	//事件监听器内部类
	private class MyItemListener implements ActionListener{
		public void actionPerformed(ActionEvent e){
			Object obj = e.getSource();
			//取得事件源
			if(obj == StartChessJFrame.this.startMenuItem || obj ==startButton){
				//重新开始
				//JFiveFrame.this //内部类引用外部类
				System.out.println("重新开始..."); 
				chessBoard.restartGame();
				repaint();
			}else if (obj == exitMenuIatem || obj == exitButton){
				System.exit(0);
				//结束应用程序
			}else if (obj == backMenuItem || obj == backButton){
				//悔棋
				System.out.println("悔棋...");
			    chessBoard.goback();
			    repaint();
			}
		}
	}
	 public static void main(String[] args){
		 StartChessJFrame f = new StartChessJFrame();
		 //创建主框架
		 f.setVisible(true);
		 //显示主框架
	 }

    
	}
