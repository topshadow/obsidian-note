# api.js

  

行数: 14189 行

  

# asc_docs_api

  

## 函数 constructor(config)

  

```javascript {.line-numbers,highlight=[1],.black}

function asc_docs_api(config) {

  AscCommon.baseEditorsApi.call(this, config, AscCommon.c_oEditorId.Word);

  

  /************ private!!! **************/

  this.WordControl = null;

  

  this.documentFormatSave = c_oAscFileType.DOCX;

  

  //todo убрать из native, copypaste, chart, loadfont

  this.InterfaceLocale = null;

  

  this.ShowParaMarks = false;

  this.ShowSmartGuides = true;

  this.isAddSpaceBetweenPrg = false;

  this.isPageBreakBefore = false;

  this.isKeepLinesTogether = false;

  

  this.isPaintFormat = c_oAscFormatPainterState.kOff;

  this.isMarkerFormat = false;

  this.isStartAddShape = false;

  this.isDrawTablePen = false;

  this.isDrawTableErase = false;

  this.addShapePreset = '';

  this.isShowTableEmptyLine = true;

  this.isShowTableEmptyLineAttack = false;

  

  this.isApplyChangesOnOpen = false;

  

  this.IsSpellCheckCurrentWord = false;

  

  this.mailMergeFileData = null;

  this.insertDocumentUrlsData = null;

  

  this.isCoMarksDraw = false;

  this.tmpCoMarksDraw = false;

  this.tmpViewRulers = null;

  this.tmpZoomType = null;

  this.tmpDocumentUnits = null;

  

  // это чтобы сразу показать ридер, без возможности вернуться в редактор/вьюер

  this.isOnlyReaderMode = false;

  

  /**************************************/

  

  this.bInit_word_control = false;

  this.isDocumentModify = false;

  

  this.tmpFontRenderingMode = null;

  this.FontAsyncLoadType = 0;

  this.FontAsyncLoadParam = null;

  

  this.isPasteFonts_Images = false;

  

  this.pasteCallback = null;

  this.pasteImageMap = null;

  this.EndActionLoadImages = 0;

  

  this.isSaveFonts_Images = false;

  this.saveImageMap = null;

  

  this.isLoadImagesCustom = false;

  this.loadCustomImageMap = null;

  

  this.ServerImagesWaitComplete = false;

  

  this.DocumentOrientation = false;

  

  this.SelectedObjectsStack = [];

  

  this.nCurPointItemsLength = -1;

  this.isDocumentEditor = true;

  

  this.CurrentTranslate = null;

  

  this.CollaborativeMarksShowType = c_oAscCollaborativeMarksShowType.All;

  

  // объекты, нужные для отправки в тулбар (шрифты, стили)

  this._gui_control_colors = null;

  

  this.DocumentReaderMode = null;

  

  if (window.editor == undefined) {

    window.editor = this;

    window['editor'] = window.editor;

  

    if (window['NATIVE_EDITOR_ENJINE']) editor = window.editor;

  }

  

  this.RevisionChangesStack = [];

  

  this.isDarkMode = false;

  

  this.TableStylesPreviewGenerator = null;

  

  //g_clipboardBase.Init(this);

  

  this._init();

}

```

  

1. 调用父类 [baseEditorsApi constructor](../common/apiBase.md)



  

问题 1.

  

- docinfo 如何初始化

  调试方法,`asc_docs_api.prototype.getInternalCoreProps`

  推测 `webapp/apps/documenteditor/main/app/view/FileMenuPanels.js`

  

```javascript

    DE.Views.FileMenuPanels.ViewSaveAs = Common.UI.BaseView.extend({

        el: '#panel-saveas',

        menu: undefined,

        setApi: function(o) {

            this.api = o;

            this.api.asc_registerCallback('asc_onGetDocInfoStart', _.bind(this._onGetDocInfoStart, this));

            this.api.asc_registerCallback('asc_onGetDocInfoStop', _.bind(this._onGetDocInfoEnd, this));

            this.api.asc_registerCallback('asc_onDocInfo', _.bind(this._onDocInfo, this));

            this.api.asc_registerCallback('asc_onGetDocInfoEnd', _.bind(this._onGetDocInfoEnd, this));

            // this.api.asc_registerCallback('asc_onDocumentName',  _.bind(this.onDocumentName, this));

            this.api.asc_registerCallback('asc_onLockCore',  _.bind(this.onLockCore, this));

            Common.NotificationCenter.on('protect:doclock', _.bind(this.onChangeProtectDocument, this));

            this.onChangeProtectDocument();

            this.updateInfo(this.doc);

            return this;

        },

        _onDocInfo: function(obj) {

            if (obj) {

                clearTimeout(this.timerLoading);

                if (obj.get_PageCount()>-1)

                    this.infoObj.PageCount = obj.get_PageCount();

                if (obj.get_WordsCount()>-1)

                    this.infoObj.WordsCount = obj.get_WordsCount();

                if (obj.get_ParagraphCount()>-1)

                    this.infoObj.ParagraphCount = obj.get_ParagraphCount();

                if (obj.get_SymbolsCount()>-1)

                    this.infoObj.SymbolsCount = obj.get_SymbolsCount();

                if (obj.get_SymbolsWSCount()>-1)

                    this.infoObj.SymbolsWSCount = obj.get_SymbolsWSCount();

                if (!this.timerDocInfo) { // start timer for filling info

                    var me = this;

                    this.timerDocInfo = setInterval(function(){

                        me.fillDocInfo();

                    }, 300);

                    this.fillDocInfo();

                }

            }

        },

            fillDocInfo:  function() {

            this.lblStatPages.text(this.infoObj.PageCount);

            this.lblStatWords.text(this.infoObj.WordsCount);

            this.lblStatParagraphs.text(this.infoObj.ParagraphCount);

            this.lblStatSymbols.text(this.infoObj.SymbolsCount);

            this.lblStatSpaces.text(this.infoObj.SymbolsWSCount);

        }

    }

```