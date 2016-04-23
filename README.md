# -*- coding: utf-8 -*-
"""Unofficial Telegram Bot API
.. _Google Python Style Guide:
   http://google.github.io/styleguide/pyguide.html
"""
from requests import Request, Session
from collections import namedtuple
from enum import Enum
from functools import partial
from abc import ABCMeta, abstractmethod
from threading import Thread
import json
"""
Telegram Bot API Types as defined at https://core.telegram
"""
_UserBase = namedtuple( 'User' , [ 'id' , 'first_name' , 'last
class User(_UserBase):
"""This object represents a Telegram user or bot.
Attributes:
id (int): Unique identifier for this user or bot
first_name (str): User‘s or bot’s first name
last_name (Optional[str]): User‘s or bot’s last na
username (Optional[str]): User‘s or bot’s username
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return User(
id=result. get( 'id' ), first_name = result. get( 'first_name' ),
last_name= result. get( 'last_name' ),
username=result. get( 'username' )
)
_GroupChatBase = namedtuple( 'GroupChat' , [ 'id' , 'title' ])
class GroupChat (_GroupChatBase):
"""This object represents a group chat.
Attributes:
id    (int): Unique identifier for this group chat
title (str): Group name
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return GroupChat(
id=result. get( 'id' ), title= result. get( 'title' )
)
_MessageBase = namedtuple( 'Message' , ['message_id' , 'sender' , 'date' , 'chat' , 'forward_fro
'document' , 'photo' , 'sticker' , 'video' , 'contact' ,
'new_chat_title' , 'new_chat_photo' , 'delete_chat_pho
class Message (_MessageBase):
"""This object represents a message.
Attributes:
message_id            (int)                 :Uniqu
from                  (User)                :Sende
date                  (int)                 :Date
chat                  (User or GroupChat)   :Conve
a priv
forward_from          (User)                :*Opti
forward_date          (int)                 :*Opti
reply_to_message      (Message)             :*Opti
text                  (str)                 :*Opti
audio                 (Audio)               :*Opti
document              (Document)            :*Opti
photo                 (Sequence[PhotoSize]) :*Opti
sticker               (Sticker)             :*Opti
video                 (Video)               :*Opti
contact               (Contact)             :*Opti
location              (Location)            :*Opti
new_chat_participant  (User)                :*Opti
left_chat_participant (User)                :*Opti
new_chat_title        (str)                 :*Opti
new_chat_photo        (Sequence[PhotoSize]) :*Opti
delete_chat_photo     (``True``)            :*Opti
group_chat_created    (``True``)            :*Opti
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
# Determine whether chat is GroupChat or User type
chat = result . get( 'chat' )
if 'id' in chat and 'title' in chat:
chat = GroupChat . from_result(chat)
else:
chat = User . from_result(chat)
# photo is a list of PhotoSize
photo = result . get( 'photo' )
if photo is not None:
photo = [PhotoSize . from_result(photo_size) for
return Message(
message_id = result. get( 'message_id' ), sender=User. from_result(result . get( 'from' )),
date=result. get( 'date' ),
chat=chat,
forward_from = User. from_result(result . get( 'forw
forward_date = result. get( 'forward_date' ),
reply_to_message = Message. from_result(result . ge
text=result. get( 'text' ),
audio= Audio . from_result(result . get( 'audio' )),
document=Document. from_result(result . get( 'docu
photo= photo,
sticker= Sticker . from_result(result . get( 'sticke
video= Video . from_result(result . get( 'video' )),
contact= Contact . from_result(result . get( 'contac
location=Location. from_result(result . get( 'loca
new_chat_participant = User.from_result(result . g
left_chat_participant = User. from_result(result .
new_chat_title = result.get( 'new_chat_title' ),
new_chat_photo = result.get( 'new_chat_photo' ),
delete_chat_photo = result. get( 'delete_chat_phot
group_chat_created = result.get( 'group_chat_crea
)
_PhotoSizeBase = namedtuple( 'PhotoSize' , [ 'file_id' , 'wid
class PhotoSize (_PhotoSizeBase):
"""This object represents one size of a photo or a fil
Attributes:
file_id    (str)  :Unique identifier for this file
width      (int)  :Photo width
height     (int)  :Photo height
file_size  (int)  :*Optional.* File size
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return PhotoSize(
file_id= result. get( 'file_id' ), width= result. get( 'width' ), height=result. get( 'height' ), file_size= result. get( 'file_size' )
)
_AudioBase = namedtuple( 'Audio' , [ 'file_id' , 'duration' ,
class Audio (_AudioBase):
"""This object represents an audio file (voice note).
Attributes:
file_id    (str)  :Unique identifier for this file
duration   (int)  :Duration of the audio in second
mime_type  (str)  :*Optional.* MIME type of the fi
file_size  (int)  :*Optional.* File size
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return Audio(
file_id= result. get( 'file_id' ), duration=result. get( 'duration' ), mime_type= result. get( 'mime_type' ), file_size= result. get( 'file_size' )
)
_DocumentBase = namedtuple( 'Document' , [ 'file_id' , 'thumb
class Document(_DocumentBase):
"""This object represents a general file (as opposed t
Attributes:
file_id    (str)        :Unique file identifier
thumb      (PhotoSize)  :Document thumbnail as def
file_name  (str)        :*Optional.* Original file
mime_type  (str)        :*Optional.* MIME type of
file_size  (int)        :*Optional.* File size
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return Document(
file_id= result. get( 'file_id' ), thumb= PhotoSize . from_result(result . get( 'thumb' file_name= result. get( 'file_name' ), mime_type= result. get( 'mime_type' ), file_size= result. get( 'file_size' ) )
_StickerBase = namedtuple( 'Sticker' , [ 'file_id' , 'width' ,
class Sticker (_StickerBase):
"""This object represents a sticker.
Attributes:
file_id    (str)        :Unique identifier for thi
width      (int)        :Sticker width
height     (int)        :Sticker height
thumb      (PhotoSize)  :Sticker thumbnail in .web
file_size  (int)        :*Optional.* File size
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return Sticker(
file_id= result. get( 'file_id' ), width= result. get( 'width' ), height=result. get( 'height' ), thumb= PhotoSize . from_result(result . get( 'thumb' file_size= result. get( 'file_size' )
)
_VideoBase = namedtuple( 'Video' , ['file_id' , 'width' , 'height' , 'duration' , 'thumb' , '
class Video (_VideoBase):
"""This object represents a video file.
Attributes:
file_id     (str)       :Unique identifier for thi
width       (int)       :Video width as defined by
height      (int)       :Video height as defined b
duration    (int)       :Duration of the video in
thumb       (PhotoSize) :Video thumbnail
mime_type   (str)       :*Optional.* Mime type of
file_size   (int)       :*Optional.* File size
caption     (str)       :*Optional.* Text descript
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return Video(
file_id= result. get( 'file_id' ), width= result. get( 'width' ), height=result. get( 'height' ), duration=result. get( 'duration' ), thumb= PhotoSize . from_result(result . get( 'thumb' mime_type= result. get( 'mime_type' ), file_size= result. get( 'file_size' ), caption= result. get( 'caption' )
)
_ContactBase = namedtuple( 'Contact' , [ 'phone_number' , 'fi
class Contact (_ContactBase):
"""This object represents a phone contact.
Attributes:
phone_number    (str)  :Contact's phone number
first_name      (str)  :Contact's first name
last_name       (str)  :*Optional.* Contact's last
user_id         (str)  :*Optional.* Contact's user
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return Contact(
phone_number = result. get( 'phone_number' ), first_name = result. get( 'first_name' ), last_name= result. get( 'last_name' ), user_id= result. get( 'user_id' )
)
_LocationBase = namedtuple( 'Location' , [ 'longitude' , 'lati
class Location(_LocationBase):
"""This object represents a point on the map.
Attributes:
longitude   (float)   :Longitude as defined by sen
latitude    (float)   :Latitude as defined by send
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
return Location(
longitude= result. get( 'longitude' ), latitude=result. get( 'latitude' )
)
_UpdateBase = namedtuple( 'Update' , [ 'update_id' , 'message
class Update(_UpdateBase):
"""This object represents an incoming update.
Attributes:
update_id   (int)     :The update‘s unique identif
positive number and increas
if you’re using Webhooks, s
restore the correct update
message     (Message) :*Optional.* New incoming me
"""
__slots__ = ()
@staticmethod
def from_dict (message_update):
if message_update is None:
return None
return Update(message_update . get( 'update_id' ), Mes
@staticmethod
def from_result (result):
if result is None:
return None
return [Update . from_dict(message_update) for messa
_InputFileInfoBase = namedtuple( 'InputFileInfo' , [ 'file_na
class InputFileInfo (_InputFileInfoBase):
__slots__ = ()
_InputFileBase = namedtuple( 'InputFile' , [ 'form' , 'file_i
class InputFile (_InputFileBase):
"""This object represents the contents of a file to be
in the usual way that files are uploaded via the
Attributes:
form        (str)           :the form used to
file_info   (InputFileInfo) :The file metadata
:Example:
::
fp = open('foo.png', 'rb')
file_info = InputFileInfo('foo.png', fp, '
InputFile('photo', file_info)
bot.send_photo(chat_id=12345678, photo=Inp
.. note::
While creating the FileInput currently req
of preparation just to send a file. This c
in the future to make the process easier.
"""
__slots__ = ()
_UserProfilePhotosBase = namedtuple( 'UserProfilePhotos' , [
class UserProfilePhotos (_UserProfilePhotosBase):
"""This object represent a user's profile pictures.
Attributes:
total_count (int): Total number of profile picture
photos      (list of list of PhotoSize): Requested
"""
__slots__ = ()
@staticmethod
def from_result (result):
if result is None:
return None
photos = []
for photo_list in result. get( 'photos' ):
photos.append([PhotoSize . from_result(photo) fo
return UserProfilePhotos(
total_count = result. get( 'total_count' ), photos=photos
)
class ReplyMarkup (metaclass =ABCMeta):
"""Abstract base to represent all valid inputs to repl
__slots__ = ()
@abstractmethod
def serialize ( self):
raise NotImplements
_ReplyKeyboardMarkupBase = namedtuple( 'ReplyKeyboardMarkup ['keyboard' , 'resize_keyboard' , 'one_time_keyboard' ,
class ReplyKeyboardMarkup (_ReplyKeyboardMarkupBase, ReplyM
"""This object represents a custom keyboard with reply
Attributes:
keyboard            (list of list of str)   :Array
resize_keyboard     (bool)  :*Optional.* Requests
fit (e.g., make th
Defaults to false,
same height as the
one_time_keyboard   (bool)  :*Optional.* Requests
used. Defaults to
selective           (bool)  :*Optional.* Use this
specific users onl
1. users that
2. if the bot'
of the orig
:example: A user r
request with a
group don’t se
:Example:
::
keyboard = [
['7', '8', '9'],
['4', '5', '6'],
['1', '2', '3'],
['0']
]
reply_markup = ReplyKeyboardMarkup.create(
bot.send_message(12345678, 'testing reply_
"""
__slots__ = ()
@staticmethod
def create(keyboard, resize_keyboard = None, one_time_ke
return ReplyKeyboardMarkup(keyboard, resize_keyboa
def serialize ( self):
reply_markup = dict(keyboard =self. keyboard)
if self. resize_keyboard is not None:
reply_markup[ 'resize_keyboard' ] = bool(self. re
if self. one_time_keyboard is not None:
reply_markup[ 'one_time_keyboard' ] = bool( self.
if self. selective is not None:
reply_markup[ 'selective' ] = bool(self. selectiv
return json. dumps(reply_markup)
_ReplyKeyboardHideBase = namedtuple( 'ReplyKeyboardHide' , [
class ReplyKeyboardHide (_ReplyKeyboardHideBase, ReplyMarku
"""Upon receiving a message with this object, Telegram
display the default letter-keyboard. By default, c
is sent by a bot. An exception is made for one-ti
user presses a button (see :class:`ReplyKeyboardMa
Attributes:
hide_keyboard   (``True``)  :Requests clients to h
selective       (bool)      :*Optional.* Use this
users only. Targets:
1. users that are
2. if the bot's me
of the original
:example: A user votes
to the vote and hi
keyboard with poll
"""
__slots__ = ()
@staticmethod
def create(selective = None):
return ReplyKeyboardHide( True, selective)
def serialize ( self):
reply_markup = dict(
hide_keyboard= True
)
if self. selective is not None:
reply_markup[ 'selective' ] = bool(self. selectiv
return json. dumps(reply_markup)
_ForceReplyBase = namedtuple( 'ForceReply' , [ 'force_reply' ,
class ForceReply(_ForceReplyBase, ReplyMarkup):
"""Upon receiving a message with this object, Telegram
(act as if the user has selected the bot‘s messag
if you want to create user-friendly step-by-step i
Attributes:
force_reply (``True``)  :Shows reply interface to
message and tapped ’Re
selective   (bool)      :Optional. Use this parame
only. Targets: 1) user
object; 2) if the bot'
sender of the original
:Example: A poll bot for groups runs in privacy mode
mentions). There could be two ways to create a new
* Explain the user how to send a command with
May be appealing for hardcore users but lack
* Guide the user through a step-by-step proces
add the first answer option‘, ’Great. Keep a
The last option is definitely more attractive. And if
receive the user’s answers even if it only receives re
work for the user.
"""
__slots__ = ()
@staticmethod
def create(selective = None):
return ForceReply( True, selective)
def serialize ( self):
reply_markup = dict(force_reply = True)
if self. selective is not None:
reply_markup[ 'selective' ] = bool(self. selectiv
return json. dumps(reply_markup)
"""
Types added for utility pruposes
"""
_ErrorBase = namedtuple( 'Error' , [ 'error_code' , 'descript
class Error (_ErrorBase):
"""The error code and message returned when a request
Attributes:
error_code  (int)   :An Integer ‘error_code’ field
contents are subject to change
description (str)   :The description of the error
"""
__slots__ = ()
@staticmethod
def from_result (result):
return Error(error_code =result. get( 'error_code' ),
"""
RPC Objects
"""
class RequestMethod ( str , Enum):
GET = 'GET'
POST = 'POST'
class TelegramBotRPCRequest (metaclass =ABCMeta):
api_url_base = 'https://api.telegram.org/bot'
def __init__( self, api_method: str , * , token, params: on_error = None, files = None, request_method
"""
:param api_method: The API method to call. See ht
:param token: The API token generated following th
https://core.telegram.org/bots#botfa
:param params: The api method parameters.
:type api_method: str
:type token: str
:type params: dict
"""
reply_markup = params .get( 'reply_markup' ) if param
if reply_markup is not None:
params['reply_markup' ] = reply_markup . serializ
self.api_method = api_method
self.token = token
self.params = params
self.on_result = on_result
self.callback = callback
self.on_error = on_error
self.files = files
self.request_method = RequestMethod(request_method
self.result = None
self.error = None
self.thread = Thread(target =self. _async_call)
def _get_url ( self):
return '{base_url}{token}/{method}' . format(base_ur
token = s
method=
def _get_request ( self):
data, files = None, None
if self. params is not None:
data = self. params
if self. files is not None:
files = self. files
return Request( self. request_method, self. _get_url(
def _async_call (self):
self.error = None
self.response = None
s = Session()
request = self. _get_request()
resp = s . send(request)
api_response = resp .json()
if api_response . get( 'ok' ):
result = api_response[ 'result' ]
if self. on_result is None:
self.result = result
else:
self.result = self. on_result(result)
if self. callback is not None:
self.callback( self. result)
else:
self.error = Error . from_result(api_response)
if self. on_error:
self.on_error( self. error)
return None
def run ( self):
self.thread. start()
return self
def join( self, timeout = None):
self.thread. join(timeout)
return self
def wait( self):
"""
Wait for the request to finish and return the resu
:returns: result or error
:type: result tyoe or Error
"""
self.thread. join()
if self. error is not None:
return self. error
return self. result
def _clean_params (**params):
return {name: val for name, val in params . items() if
def _merge_user_overrides (request_args, **kwargs):
result = request_args .copy() if request_args is not No
if kwargs is not None:
result.update(kwargs)
return result
"""
Telegram Bot API Methods as defined at https://core.telegr
"""
def get_me ( *, request_args =None, **kwargs):
"""
A simple method for testing your bot's auth token. Re
Returns basic information about the bot in form of a
:param request_args: Args passed down to TelegramBotRP
:returns: Returns basic information about the bot in
:rtype: User
"""
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'getMe' , on_result = User.f
def send_message (chat_id: int , text: str , disable_web_page_preview: bool= None, repl reply_markup: ReplyMarkup = None,
* , request_args = None, **kwargs):
"""
Use this method to send text messages.
:param chat_id: Unique identifier for the message reci
:param text: Text of the message to be sent
:param disable_web_page_preview: Disables link preview
:param reply_to_message_id: If the message is a reply
:param reply_markup: Additional interface options. A J
custom reply keyboard, instructio
force a reply from the user.
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type text: str
:type disable_web_page_preview: bool
:type reply_to_message_id: int
:type reply_markup: :class:`ReplyKeyboardMarkup` or :c
:returns: On success, the sent Message is returned.
:rtype: Message
"""
# required args
params = dict(chat_id =chat_id, text = text)
# optional args
params.update(_clean_params(
disable_web_page_preview = disable_web_page_preview,
reply_to_message_id =reply_to_message_id,
reply_markup = reply_markup
)
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendMessage' , params = par
def forward_message (chat_id, from_chat_id, message_id,
*, request_args =None, **kwargs):
"""
Use this method to forward messages of any kind.
:param chat_id: Unique identifier for the message reci
:param from_chat_id: Unique identifier for the chat wh
GroupChat id
:param message_id: Unique message identifier
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type from_chat_id: int
:type message_id: int
:returns: On success, the sent Message is returned.
:rtype: Message
"""
# required args
params = dict(
chat_id= chat_id, from_chat_id = from_chat_id, message_id = message_id
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'forwardMessage' , params =
def send_photo (chat_id: int ,  photo: InputFile, caption: str =None, reply_to_message_id: int
* , request_args = None, **kwargs) -> Telegram
"""
Use this method to send photos.
:param chat_id: Unique identifier for the message reci
:param photo: Photo to send. You can either pass a fi
photo that is already on the Telegram se
using multipart/form-data.
:param caption: Photo caption (may also be used when
:param reply_to_message_id: If the message is a reply
:param reply_markup: Additional interface options. A J
custom reply keyboard, instructio
force a reply from the user.
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type photo: InputFile or str
:type caption: str
:type reply_to_message_id: int
:type reply_markup: :class:`ReplyKeyboardMarkup` or :c
:returns: On success, the sent Message is returned.
:rtype: TelegramBotRPCRequest
"""
files = None
if isinstance(photo, InputFile):
files = [photo]
photo = None
elif not isinstance(photo, str ):
raise Exception ( 'photo must be instance of InputFi
# required args
params = dict(
chat_id= chat_id,
photo= photo
)
# optional args
params.update(
_clean_params(
caption= caption,
reply_to_message_id =reply_to_message_id,
reply_markup = reply_markup
)
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendPhoto' , params = param
**request_args) . run()
def send_audio (chat_id: int , audio: InputFile, reply_to_m
reply_markup: ReplyKeyboardMarkup = None,
* , request_args = None, **kwargs) -> Telegram
"""
Use this method to send audio files, if you want Teleg
message. For this to work, your audio must be in an .
as Document).
:param chat_id: Unique identifier for the message reci
:param audio: Audio file to send. You can either pass
the Telegram servers, or upload a new au
:param reply_to_message_id: If the message is a reply
:param reply_markup: Additional interface options. A J
instructions to hide keyboard or
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type audio: InputFile or str
:type reply_to_message_id: int
:type reply_markup: ReplyKeyboardMarkup or ReplyKeyboa
:returns: On success, the sent Message is returned.
:rtype: TelegramBotRPCRequest
"""
files = None
if isinstance(audio, InputFile):
files = [audio]
audio = None
elif not isinstance(audio, str ):
raise Exception ( 'audio must be instance of InputFi
# required args
params = dict(
chat_id= chat_id,
audio= audio
)
# optional args
params.update(
_clean_params(
reply_to_message_id =reply_to_message_id,
reply_markup = reply_markup
)
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendAudio' , params = param
**request_args) . run()
def send_document (chat_id: int , document: InputFile, reply
reply_markup: ReplyKeyboardMarkup =None,
*, request_args =None, **kwargs) -> Teleg
"""
Use this method to send general files.
:param chat_id: Unique identifier for the message reci
:param document: File to send. You can either pass a
the Telegram servers, or upload a new
:param reply_to_message_id: If the message is a reply
:param reply_markup: Additional interface options. A J
instructions to hide keyboard or
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type document: InputFile or str
:type reply_to_message_id: int
:type reply_markup: ReplyKeyboardMarkup or ReplyKeyboa
:returns: On success, the sent Message is returned.
:rtype: TelegramBotRPCRequest
"""
files = None
if isinstance(document, InputFile):
files = [document]
document = None
elif not isinstance(document, str ):
raise Exception ( 'document must be instance of Inpu
# required args
params = dict(
chat_id= chat_id,
document=document
)
# optional args
params.update(
_clean_params(
reply_to_message_id =reply_to_message_id,
reply_markup = reply_markup
)
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendDocument' , params = pa
**request_args) . run()
def send_sticker (chat_id: int , sticker: InputFile, reply_
reply_markup: ReplyKeyboardMarkup = None,
* , request_args = None, **kwargs) -> Telegr
"""
:param chat_id: Unique identifier for the message reci
:param sticker: Sticker to send. You can either pass
that is already on the Telegram server
:param reply_to_message_id: If the message is a reply
:param reply_markup: Additional interface options. A J
instructions to hide keyboard or
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type sticker: InputFile or str
:type reply_to_message_id: int
:type reply_markup: ReplyKeyboardMarkup or ReplyKeyboa
:returns: On success, the sent Message is returned.
:rtype: TelegramBotRPCRequest
"""
files = None
if isinstance(sticker, InputFile):
files = [sticker]
sticker = None
elif not isinstance(sticker, str ):
raise Exception ( 'sticker must be instance of Input
# required args
params = dict(
chat_id= chat_id,
sticker= sticker
)
# optional args
params.update(
_clean_params(
reply_to_message_id =reply_to_message_id,
reply_markup = reply_markup
)
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendSticker' , params = par
**request_args) . run()
def send_video (chat_id: int , video: InputFile, reply_to_m
reply_markup: ReplyKeyboardMarkup = None,
* , request_args = None, **kwargs) -> Telegram
"""
Use this method to send video files, Telegram clients
:param chat_id: Unique identifier for the message reci
:param video: Video to send. You can either pass a fi
video that is already on the Telegram se
using multipart/form-data.
:param reply_to_message_id: If the message is a reply
:param reply_markup: Additional interface options. A J
custom reply keyboard, instructio
force a reply from the user.
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type video: InputFile or str
:type reply_to_message_id: int
:type reply_markup: ReplyKeyboardMarkup or ReplyKeyboa
:returns: On success, the sent Message is returned.
:rtype:  TelegramBotRPCRequest
"""
files = None
if isinstance(video, InputFile):
files = [video]
video = None
elif not isinstance(video, str ):
raise Exception ( 'video must be instance of InputFi
# required args
params = dict(
chat_id= chat_id,
video= video
)
# optional args
params.update(
_clean_params(
reply_to_message_id =reply_to_message_id,
reply_markup = reply_markup
)
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendVideo' , params = param
**request_args) . run()
def send_location (chat_id: int , latitude: float , longitude
reply_markup: ReplyKeyboardMarkup =None,
*, request_args =None, **kwargs) -> Teleg
"""
Use this method to send point on the map.
:param chat_id: Unique identifier for the message reci
:param latitude: Latitude of location.
:param longitude: Longitude of location.
:param reply_to_message_id: If the message is a reply
:param reply_markup: Additional interface options. A J
custom reply keyboard, instructio
force a reply from the user.
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type latitude: float
:type longitude: float
:type reply_to_message_id: int
:type reply_markup: ReplyKeyboardMarkup or ReplyKeyboa
:returns: On success, the sent Message is returned.
:rtype:  TelegramBotRPCRequest
"""
# required args
params = dict(
chat_id= chat_id,
latitude=latitude,
longitude= longitude
)
# optional args
params.update(
_clean_params(
reply_to_message_id =reply_to_message_id,
reply_markup = reply_markup
)
)
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendLocation' , params = pa
**request_args) . run()
class ChatAction( str , Enum):
TEXT = 'typing'
PHOTO = 'upload_photo'
RECORD_VIDEO = 'record_video'
VIDEO = 'upload_video'
RECORD_AUDIO = 'record_audio'
AUDIO = 'upload_audio'
DOCUMENT = 'upload_document'
LOCATION = 'find_location'
def send_chat_action (chat_id: int , action: ChatAction,
* , request_args = None, **kwargs) -> Te
"""
Use this method when you need to tell the user that so
for 5 seconds or less (when a message arrives from y
Example: The ImageBot needs some time to process a req
along the lines of “Retrieving image, please wait…”, t
The user will see a “sending photo” status for the bo
We only recommend using this method when a response fr
:param chat_id: Unique identifier for the message reci
:param action: Type of action to broadcast.  Choose on
typing for text messages, upload_photo
record_audio or upload_audio for audio
find_location for location data.
:param request_args: Args passed down to TelegramBotRP
:type chat_id: int
:type action: ChatAction
:returns: Returns True on success.
:rtype:  bool
"""
# required args
params = dict(
chat_id= chat_id,
action=action
)
# merge bot args with user overrides
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'sendChatAction' , params =
def get_user_profile_photos (user_id: int , offset: int = Non
*, request_args: dict= None, **
"""
Use this method to get a list of profile pictures for
:param user_id: Unique identifier of the target user
:param offset: Sequential number of the first photo to
:param limit: Limits the number of photos to be retrie
:param request_args: Args passed down to the TelegramB
:type user_id: int
:type offset: int
:type limit: int
:returns: Returns a UserProfilePhotos object.
:rtype: TelegramBotRPCRequest
"""
# required args
params = dict(user_id =user_id)
# optional args
params.update(
_clean_params(
offset=offset,
limit= limit
)
)
# merge bot args with user overrides
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'getUserProfilePhotos' , p on_result = UserProfilePhot
def get_updates (offset: int = None, limit: int= None, timeou
*, request_args, **kwargs):
"""
Use this method to receive incoming updates using long
.. note::
1. This method will not work if an outgoing webhoo
2. In order to avoid getting duplicate updates, re
:param offset: Identifier of the first update to be re
greater by one than the highest among t
previously received updates. By default
with the earliest unconfirmed update ar
is considered confirmed as soon as getU
an offset higher than its update_id.
:param limit: Limits the number of updates to be retri
1—100 are accepted. Defaults to 100
:param timeout: Timeout in seconds for long polling.
usual short polling
:param request_args: Args passed down to the TelegramB
:param kwargs: Args passed down to the TelegramBotRPCR
:type offset: int
:type limit: int
:type timeout: int
:returns: An Array of Update objects is returned.
:rtype: TelegramBotRPCRequest
"""
# optional parameters
params = _clean_params(
offset=offset,
limit= limit,
timeout= timeout
)
# merge bot args with user overrides
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'getUpdates' , params = para
def set_webhook (url: str =None,
*, request_args =None, **kwargs) -> Telegra
"""
Use this method to specify a url and receive incoming
Whenever there is an update for the bot, we will send
JSON-serialized Update. In case of an unsuccessful req
Please note that you will not be able to receive updat
webhook is set up.
:param url: HTTPS url to send updates to. Use an empt
:param request_args: Args passed down to TelegramBotRP
:type url: str
:returns: Returns True on success.
:rtype:  TelegramBotRPCRequest
"""
# optional args
params = _clean_params(url = url)
# merge bot args with user overrides
request_args = _merge_user_overrides(request_args, **k
return TelegramBotRPCRequest( 'setWebhook' , params = para
on_result = lambda result:
class TelegramBot :
def __init__( self, token, request_method: RequestMetho
self._bot_user = None
self.request_args = dict(
token= token,
request_method = request_method
)
self.get_me = partial(get_me, request_args = self.re
self.send_message = partial(send_message, request_
self.forward_message = partial(forward_message, re
self.send_photo = partial(send_photo, request_args
self.send_audio = partial(send_audio, request_args
self.send_document = partial(send_document, reques
self.send_sticker = partial(send_sticker, request_
self.send_video = partial(send_video, request_args
self.send_location = partial(send_location, reques
self.send_chat_action = partial(send_chat_action,
self.get_user_profile_photos = partial(get_user_pr
self.get_updates = partial(get_updates, request_ar
self.set_webhook = partial(set_webhook, request_ar
self.update_bot_info = partial(get_me, request_arg
def __str__ ( self):
return self. token
@property
def token ( self):
return self. request_args[ 'token' ]
@token.setter
def token ( self, val):
self.request_args[ 'token' ] = val
@property
def request_method ( self):
return self. request_args[ 'request_method' ]
@request_method.setter
def request_method ( self, val: RequestMethod):
self.request_args[ 'request_method' ] = val
@property
def id( self):
if self. _bot_user is not None:
return self. _bot_user .id
@property
def first_name ( self):
if self. _bot_user is not None:
return self. _bot_user .first_name
@property
def last_name ( self):
if self. _bot_user is not None:
return self. _bot_user .last_name
@property
def username( self):
if self. _bot_user is not None:
return self. _bot_user .username
def _update_bot_info ( self, bot_user):
self._bot_user = bot_user
