# The Universal Metadata Format

This is the specification for the UMF (Universal Metadata Format). UMF is a standard for storing information about a piece of media. This specification defines the basic syntax and structure for the format, along with the required headers and fields for each media type.

## Sections

- [Basic Syntax & Structure](#basic-syntax--structure)
- [Naming Style & Spacing](#naming-style--spacing)
- [Parsing Details & Errors](#parsing-details--errors)
- [Media Types & Presets](#media-types--presets)
  - [Anime](#anime)
  - [Music](#music)

## Implementataions

- [TypeScript](https://github.com/shmg-org/umf-typescript)
- [Zig](https://github.com/shmg-org/umf-zig)

## Basic Syntax & Structure

A metadata consist of a media name, headers, and fields. The first line of the metadata must always be the media name, followed by fields or headers. A header defines the scope of the fields below it, and a field is considered global if it is not under any header. If a line starts with "#" it is considered as a comment.

- An example with a header and fields.

```none
<Media Name>

<Field Name>: <Field Value>
<Field Name>: <Field Value>

[ <Header> ]

<Field Name>: <Field Value>
<Field Name>: <Field Value>
```

- An example with global fields and a comment.

```none
<Media Name>

<Field Name>: <Field Value>
<Field Name>: <Field Value>

# This is a comment.
```

> [!NOTE]
> Fields that are under a header have higher priority than global fields, global fields are only used as a fallback.

## Naming Style & Spacing

When defining a header or field, the first character in the name must always be capitalized. It is recommended to have an empty line after the media name, a space around the header name, and a space after the ":" character in a field. But those are not requirements, you can format them as you like.

- An example with the recommended formatting.

```none
The name of the media

[ A Header ]

Field1: Value1
Field2: Value2
```

- An example with the not-recommended formatting.

```none
The name of the media
[Header 1]
Field1:Value1
Field2:Value2
```

## Parsing Details & Errors

UMF is a line by line format, meaning the base unit of parsing is a line. Before parsing a line, the parser must remove the spaces at the start and the end of the line. The first line of a metadata is always the media name, the lines after can be parsed using the following rules, the parser must check the line against each condition in order.

1. If the line is empty or start with "#", ignore the line.
2. If the line starts with "\[" and end with "\]" it is a header.
3. If the line includes a ":" it is a field.

If no rules can apply to the line, throw an error. Otherwise, keep parsing the line using the respective rule:

- **Header**: Exclude the starting "\[" and the ending "\]", and trim the result.
- **Field**: Split the line with the first ":" and trim all the parts, the first part is the name, the second part is the value.

### Errors

The parser can throw errors based on it's custom formatting rules, but the following errors are required to thrown when the specified condition is met:

- **Empty Media Name**: When the media name is empty or not found.
- **Empty Header Name**: When the name of a header is empty.
- **Empty Field Name**: When the name of a field is empty.
- **Empty Field Value**: When the value of a field is empty.
- **Invalid Line**: When the line cannot be parsed.

## Media Types & Presets

UMF provides some presets for how the metadata of a media type should be defined. All the headers are fields defined in this specification are required, but extra ones can be added for your personal use.

> [!NOTE]
> These presets should be implemented on the apllication side, meaning applications that uses those presets need to implement and follow this specificatoin themself.

### Anime

The anime preset uses headers to add season specific metadata. If there are fields that stay the same across all seasons, it's recommended to define them as global fields. The metadata must include the following fields:

- Type: The type of the source video.
  - Accepted type: `BDRip` or `WebRip`.
- Source: The source of the video (encoder, platform, etc...).
  - Formatted as: `<Source Name> (<Source URL?>)`
- Subtitle: The source of the subtitle (subtitle team, platform, etc...).
  - Formatted as: `<Source Name> (<Source URL?>)`
- Resolution: The resolution of the video.
  - Accepted resolution can be `720p`, `1080p`, `1440p`, or `4K`.
- Encoding: The video and audio codec for the anime.
  - Accepted video codec: `H.264`, `H.265`, or `VP9`.
  - Accepted audio codec: `AAC` or `FLAC`.
  - Formatted as: `<Video Codec> (<Audio Codec>)`

### Music

The music preset uses headers to add track specific metadata. If there are fields that stay the same across all seasons, it's recommended to define them as global fields. The metadata must include the following fields:

- Type: The type of the source audio.
  - Accepted type: `CDRip`, `GameRip`, or `WebRip`.
- Performer: The performer of the track (optional).
- Composer: The compose of the track (optional).
- Arranger: The arranger of the track (optional).
- Lyricist: The lyricist of the track (optional).
- Track Number: The index of the track in the album.
