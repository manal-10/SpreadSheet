# SpreadSheet
# SpreadSheet.cpp
#include "spreadsheet.h"
#include <QPixmap>
#include <QMenuBar>
#include <QToolBar>
#include <QApplication>
#include <QMessageBox>
#include "godialog.h"
#include "finddialog.h"
#include <QFileDialog>
#include <QTextStream>
#include <QClipboard>


SpreadSheet::SpreadSheet(QWidget *parent)
    : QMainWindow(parent)
{
    //Seting the spreadsheet

    setupMainWidget();

    // Creaeting Actions
    createActions();

    // Creating Menus
    createMenus();


    //Creating the tool bar
    createToolBars();

    //making the connexions
    makeConnexions();


    //Creating the labels for the status bar (should be in its proper function)
    cellLocation = new QLabel("(1, 1)");
    cellFormula = new QLabel("");
    statusBar()->addPermanentWidget(cellLocation);
    statusBar()->addPermanentWidget(cellFormula);

    currentFile = nullptr;
    setWindowTitle("Buffer");

    //changer les labels du spreadSheet
    QStringList labels;
    //Remplir la liste des labels
    for(char letter = 'A'; letter<= 'Z'; letter++)
        labels <<QString(letter);
    spreadsheet->setVerticalHeaderLabels(labels);
}

void SpreadSheet::setupMainWidget()
{
    spreadsheet = new QTableWidget;
    spreadsheet->setRowCount(100);
    spreadsheet->setColumnCount(12);
    setCentralWidget(spreadsheet);

}

SpreadSheet::~SpreadSheet()
{
    delete spreadsheet;

    // --------------- Actions       --------------//
    delete  newFile;
    delete  open;
    delete  save;
    delete  saveAs;
    delete  exit;
    delete cut;
    delete copy;
    delete paste;
    delete deleteAction;
    delete find;
    delete row;
    delete column;
    delete all;
    delete goCell;
    delete recalculate;
    delete sort;
    delete showGrid;
    delete auto_recalculate;
    delete about;
    delete aboutQt;

    // ---------- Menus ----------
    delete FileMenu;
    delete editMenu;
    delete toolsMenu;
    delete optionsMenu;
    delete helpMenu;
}

void SpreadSheet::createActions()
{
    // --------- New File -------------------
   QPixmap newIcon(":/new_file.png");
   newFile = new QAction(newIcon, "&New", this);
   newFile->setShortcut(tr("Ctrl+N"));


    // --------- open file -------------------
   open = new QAction("&Open", this);
   open->setShortcut(tr("Ctrl+O"));

    // --------- save file -------------------
   QPixmap saveIcon(":/save_file.png");
   save = new QAction(saveIcon, "&Save", this);
   save->setShortcut(tr("Ctrl+S"));

    // --------- save as file -------------------
   saveAs = new QAction(saveIcon, "save &As", this);


    // --------- cut file -------------------
   QPixmap cutIcon(":/cut_icon.png");
   cut = new QAction(cutIcon, "Cu&t", this);
   cut->setShortcut(tr("Ctrl+X"));

   // --------- Copy menu -----------------
   copy = new QAction( "&Copy", this);
   copy->setShortcut(tr("Ctrl+C"));


   paste = new QAction( "&Paste", this);
   paste->setShortcut(tr("Ctrl+V"));


   deleteAction = new QAction( "&Delete", this);
   deleteAction->setShortcut(tr("Del"));


   row  = new QAction("&Row", this);
   column = new QAction("&Column", this);
   all = new QAction("&All", this);
   all->setShortcut(tr("Ctrl+A"));



   QPixmap findIcon(":/search_icon.png");
   find= new QAction(findIcon, "&Find", this);
   find->setShortcut(tr("Ctrl+F"));

   QPixmap goCellIcon(":/go_to_icon.png");
   goCell = new QAction( goCellIcon, "&Go to Cell", this);
   deleteAction->setShortcut(tr("f5"));


   recalculate = new QAction("&Recalculate",this);
   recalculate->setShortcut(tr("F9"));


   sort = new QAction("&Sort");



   showGrid = new QAction("&Show Grid");
   showGrid->setCheckable(true);
   showGrid->setChecked(spreadsheet->showGrid());

   auto_recalculate = new QAction("&Auto-recalculate");
   auto_recalculate->setCheckable(true);
   auto_recalculate->setChecked(true);



   about =  new QAction("&About");
   aboutQt = new QAction("About &Qt");

    // --------- exit -------------------
   QPixmap exitIcon(":/quit_icon.png");
   exit = new QAction(exitIcon,"E&xit", this);
   exit->setShortcut(tr("Ctrl+Q"));
}

void SpreadSheet::close()
{

    auto reply = QMessageBox::question(this, "Exit",
                                       "Do you really want to quit?");
    if(reply == QMessageBox::Yes)
        qApp->exit();
}

void SpreadSheet::createMenus()
{
    // --------  File menu -------//
    FileMenu = menuBar()->addMenu("&File");
    FileMenu->addAction(newFile);
    FileMenu->addAction(open);
    FileMenu->addAction(save);
    FileMenu->addAction(saveAs);
    FileMenu->addSeparator();
    FileMenu->addAction(exit);


    //------------- Edit menu --------/
    editMenu = menuBar()->addMenu("&Edit");
    editMenu->addAction(cut);
    editMenu->addAction(copy);
    editMenu->addAction(paste);
    editMenu->addAction(deleteAction);
    editMenu->addSeparator();
    auto select = editMenu->addMenu("&Select");
    select->addAction(row);
    select->addAction(column);
    select->addAction(all);

    editMenu->addAction(find);
    editMenu->addAction(goCell);



    //-------------- Toosl menu ------------
    toolsMenu = menuBar()->addMenu("&Tools");
    toolsMenu->addAction(recalculate);
    toolsMenu->addAction(sort);



    //Optins menus
    optionsMenu = menuBar()->addMenu("&Options");
    optionsMenu->addAction(showGrid);
    optionsMenu->addAction(auto_recalculate);


    //----------- Help menu ------------
    helpMenu = menuBar()->addMenu("&Help");
    helpMenu->addAction(about);
    helpMenu->addAction(aboutQt);
}

void SpreadSheet::createToolBars()
{

    //Crer une bare d'outils
    auto toolbar1 = addToolBar("File");


    //Ajouter des actions acette bar
    toolbar1->addAction(newFile);
    toolbar1->addAction(save);
    toolbar1->addSeparator();
    toolbar1->addAction(exit);


    //Creer une autre tool bar
    auto toolbar2  = addToolBar("ToolS");
    toolbar2->addAction(goCell);
    toolbar2->addAction(find);
}

void SpreadSheet::updateStatusBar(int row, int col)
{
    QString cell{"(%0, %1)"};
   cellLocation->setText(cell.arg(row+1).arg(col+1));

}

void SpreadSheet::makeConnexions()
{

   // --------- Connexion for the  select all action ----/
   connect(all, &QAction::triggered,
           spreadsheet, &QTableWidget::selectAll);

   // Connection for the  show grid
   connect(showGrid, &QAction::triggered,
           spreadsheet, &QTableWidget::setShowGrid);

   //Connection for the exit button
   connect(exit, &QAction::triggered, this, &SpreadSheet::close);


   //connectting the chane of any element in the spreadsheet with the update status bar
   connect(spreadsheet, &QTableWidget::cellClicked, this,  &SpreadSheet::updateStatusBar);

   //Connexion pour le goDialog
   connect(goCell, &QAction::triggered, this, &SpreadSheet::goCellSlot);

   //Connection pour le Search
   connect(find, &QAction::triggered, this, &SpreadSheet::findCellSlot);

   connect(save, &QAction::triggered, this, &SpreadSheet::saveSlot);

   connect(open, &QAction::triggered, this, &SpreadSheet::openSlot);
   //
   connect(copy, &QAction::triggered, this, &SpreadSheet::copySlot);

   connect(paste, &QAction::triggered, this, &SpreadSheet::pasteSlot);

   connect(row, &QAction::triggered, this, &SpreadSheet::rowSelect);

   connect(column, &QAction::triggered, this, &SpreadSheet::colSelect);

   connect(deleteAction, &QAction::triggered, this, &SpreadSheet::deleteFunction);

   connect(open, &QAction::triggered, this, &SpreadSheet::openCSV);

}

void SpreadSheet::goCellSlot()
{
    //1. Creer le dialog
    goDialog D;

    //2. Executer le dialog
    auto reply =D.exec();

    //3. Verifier si le dialog a ete accepte
    if(reply == goDialog::Accepted)
        //QMessageBox::information(this, "Go", "Accepted")
    {
        QString cell = D.getCell();

        //Extraire la ligne
        int row = cell[0].toLatin1() - 'A';


        cell = cell.remove(0, 1);
        int col = cell.toInt()-1;


        QStatusBar().showMessage("Chanding the current cell", 2000);
        spreadsheet->setCurrentCell(row, col);
    }
}

void SpreadSheet::findCellSlot()
{
     findDialog dialog;

    auto reply = dialog.exec();
    auto found = false;

    if(reply == findDialog::Accepted)
    {
        QString text = dialog.getText();
        for(auto i=0; i<spreadsheet->rowCount(); i++)
        {
            for(auto j=0; j<spreadsheet->columnCount(); j++)
            {
                QString searchWord = spreadsheet->item(i,j)->text();
            }
        }
    }*/

    findDialog f;

     //auto reply =  f.exec();

     //extraire le text
     QString cell =f.getCell();
     int row = spreadsheet->rowCount();
     int col = spreadsheet->columnCount();

     for (int i=0;i<row ;i++){
        for (int j=0;j<col ;j++){
           auto location = spreadsheet->item(i,j);
           if(location && location->text()==cell){
               spreadsheet->setCurrentCell(i, j);
               return;
        }
     }
   }
}

void SpreadSheet::saveSlot()
{
    //1. Tester si on possede un nom de fichier
    if(!currentFile)
    {
        QFileDialog D;

        auto filename = D.getSaveFileName();
        currentFile = new QString(filename);
        setWindowTitle(*currentFile);
    }

    //Sauvegarder le contenu
    saveContent(*currentFile);
}

void SpreadSheet::saveContent(QString filename) const
{
    //suntaxe c++
    //ofstream out(filename)---- ./ out.close();

    //suntaxe c
    // FILE * fid = fOpen(filename, "w")   fclose(fid);

    // en QT
    //1. Pointeur sur le fichier d'interet
    QFile file(filename);

    if( file.open(QIODevice::WriteOnly))
    {
        QTextStream out(&file);

        for(int i=0; i<spreadsheet->rowCount(); i++)
            for(int j=0; j< spreadsheet->columnCount(); j++)
            {
                auto cell = spreadsheet->item(i, j);
                if(cell)
                {
                    out<< i << "," << j << "," << cell->text() ;//<< endl
                }
            }
    }

    file.close();
}

void SpreadSheet::openSlot()
{
    QFileDialog D;

    auto filename = D.getOpenFileName();
    if(filename != "")
    {
        currentFile = new QString(filename);
        setWindowTitle(filename);
        loadContent(filename);
    }

}

void SpreadSheet::loadContent(QString filename)
{
    QFile file(filename);

    if(file.open(QIODevice::ReadOnly))
    {
        QTextStream in(&file);
        QString line;

        while( !in.atEnd())
        {
            line = in.readLine();
            auto tokens = line.split(QChar(','));
            int row = tokens[0].toInt();
            int col = tokens[1].toInt();
            spreadsheet->setItem(row, col, new QTableWidgetItem(tokens[2]));
        }
    }
}

void SpreadSheet::copySlot()
{
    QApplication::clipboard()->setText(spreadsheet->currentIndex().data().toString());
}

void SpreadSheet::pasteSlot()
{

}
void SpreadSheet::rowSelect()
{
    spreadsheet->selectRow(spreadsheet->currentRow());
}

void SpreadSheet::colSelect()
{
    spreadsheet->selectColumn(spreadsheet->currentColumn());
}

void SpreadSheet::deleteFunction()
{
    spreadsheet->currentItem()->setText(NULL);
}




