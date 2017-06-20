# Parser/Generator m3u playlists.

[![Build Status](https://secure.travis-ci.org/Gemorroj/M3uParser.png?branch=master)](https://travis-ci.org/Gemorroj/M3uParser)


### Requirements:

- PHP >= 5.6


### Installation:

- Add to composer.json:

```json
{
    "require": {
        "gemorroj/m3u-parser": "*"
    }
}
```
- install project:

```bash
$ composer update gemorroj/m3u-parser
```


### Example parser:

```php
<?php
use M3uParser\M3uParser;

$m3uParser = new M3uParser();
$data = $m3uParser->parseFile('path_to_file.m3u');

print_r($data->getAttributes());
/*
Array
(
    [url-tvg] => http://www.teleguide.info/download/new3/jtv.zip
    [m3uautoload] => 1
    [deinterlace] => 8
    [cache] => 500
)
*/

/** @var \M3uParser\Entry $entry */
foreach ($data as $entry) {
    print_r($entry);
    /*
        M3uParser\Entry Object
        (
            [lineDelimiter:protected] =>

            [extTags:M3uParser\Entry:private] => Array
                (
                    [0] => M3uParser\Tag\ExtInf Object
                        (
                            [title:M3uParser\Tag\ExtInf:private] => TV SLO 1 HD
                            [duration:M3uParser\Tag\ExtInf:private] => 1
                            [attributes:M3uParser\Tag\ExtInf:private] => Array
                                (
                                )

                        )

                    [1] => M3uParser\Tag\ExtTv Object
                        (
                            [tags:M3uParser\Tag\ExtTv:private] => Array
                                (
                                    [0] => Slovenski
                                    [1] => HD
                                )

                            [language:M3uParser\Tag\ExtTv:private] => slv
                            [xmlTvId:M3uParser\Tag\ExtTv:private] => SLO1HD
                            [iconUrl:M3uParser\Tag\ExtTv:private] =>
                        )

                )

            [path:M3uParser\Entry:private] => rtp://@232.2.201.53:5003
        )
    */

    echo $entry->getPath() . "\n";

    foreach ($entry->getExtTags() as $extTag) {
        switch ($extTag) {
            case $extTag instanceof \M3uParser\Tag\ExtInf: // If EXTINF tag
                echo $extTag->getTitle() . "\n";
                echo $extTag->getDuration() . "\n";

                if ($extTag->getAttribute('tvg-name')) { // If tvg-name attribute in EXTINF tag
                    echo $extTag->getAttribute('tvg-name') . "\n";
                }
                break;

            case $extTag instanceof \M3uParser\Tag\ExtTv: // If EXTTV tag
                echo $extTag->getXmlTvId() . "\n";
                echo $extTag->getIconUrl() . "\n";
                echo $extTag->getLanguage() . "\n";
                foreach ($extTag->getTags() as $tag) {
                    echo $tag . "\n";
                }
                break;
        }
    }
}
```

### Example generator:

```php
<?php
use M3uParser\Data;
use M3uParser\Entry;
use M3uParser\Tag\ExtInf;
use M3uParser\Tag\ExtTv;

$entry = new Entry();
$entry->setPath('test-path');
$entry->addExtTag(
    (new ExtInf())
        ->setDuration(123)
        ->setTitle('extinf-title')
        ->setAttribute('test-attr', 'test-attrname')
);
$entry->addExtTag(
    (new ExtTv())
        ->setIconUrl('https://example.org/icon.png')
        ->setLanguage('ru')
        ->setXmlTvId('xml-tv-id')
        ->setTags(['hd', 'sd'])
);

$data = new Data();
$data->setAttribute('test-name', 'test-value');
$data->append($entry);

echo $data;
/*
#EXTM3U test-name="test-value"
#EXTINF: 123 test-attr="test-attrname", extinf-title
#EXTTV: hd,sd;ru;xml-tv-id;https://example.org/icon.png
test-path
*/
```

### Example custom tag:
```
#EXTM3U
#EXTCUSTOMTAG:123
http://nullwave.barricade.lan:8000/club
```

```php
<?php

use M3uParser\M3uParser;
use M3uParser\Tag\ExtTagInterface;

class ExtCustomTag implements ExtTagInterface
{
    /**
     * @var string
     */
    private $data;

    /**
     * #EXTCUSTOMTAG:data
     * @param string $lineStr
     */
    public function __construct($lineStr = null)
    {
        if (null !== $lineStr) {
            $this->makeData($lineStr);
        }
    }

    /**
     * @param string $lineStr
     */
    protected function makeData($lineStr)
    {
        /*
EXTCUSTOMTAG format:
#EXTCUSTOMTAG:data
example:
#EXTCUSTOMTAG:123
         */

        $data = \substr($lineStr, \strlen('#EXTCUSTOMTAG:'));

        $this->setData(\trim($data));
    }

    /**
     * @return string
     */
    public function getData()
    {
        return $this->data;
    }

    /**
     * @param string $data
     * @return $this
     */
    public function setData($data)
    {
        $this->data = $data;
        return $this;
    }

    /**
     * @return string
     */
    public function __toString()
    {
        return '#EXTCUSTOMTAG: ' . $this->getData();
    }

    /**
     * @param string $lineStr
     * @return bool
     */
    public static function isMatch($lineStr)
    {
        return '#EXTCUSTOMTAG:' === \strtoupper(\substr($lineStr, 0, \strlen('#EXTCUSTOMTAG:')));
    }
}


$m3uParser = new M3uParser([ExtCustomTag::class]);
$data = $m3uParser->parseFile('path_to_file.m3u');

print_r($data);
/*
M3uParser\Data Object
(
    [attributes:M3uParser\Data:private] => Array
        (
        )

    [storage:ArrayIterator:private] => Array
        (
            [0] => M3uParser\Entry Object
                (
                    [lineDelimiter:protected] =>

                    [extTags:M3uParser\Entry:private] => Array
                        (
                            [0] => M3uParser\Tests\ExtCustomTag Object
                                (
                                    [data:M3uParser\Tests\ExtCustomTag:private] => 123
                                )

                        )

                    [path:M3uParser\Entry:private] => http://nullwave.barricade.lan:8000/club
                )

        )

)
*/
```
